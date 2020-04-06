Redis 2.6 脚本功能(scripting)简介
========================================


导读
------------

对 Lua 脚本的支持无疑是 Redis 2.6 版本的最大亮点，本文通过一个 ``LPOPRPUSH`` 操作为例子，介绍如何使用 ``EVAL`` 命令对 Lua 脚本进行求值，以及如何在 Lua 脚本中执行 Redis 命令，并说明这些新特性又是怎样漂亮地解决一些 Redis 在 2.6 版本前很难解决的问题的。


LPOPRPUSH
------------

Redis 的 ``RPOPLPUSH`` 命令有一个特点：它是 Redis 列表结构的众多命令里，唯一一个左右不对称的命令 —— 在 Redis 里，只有 ``RPOPLPUSH`` ，却没有 ``LPOPRPUSH`` ，而其他列表结构的命令，比如 ``LPUSH`` 和 ``RPUSH`` 、 ``LPUSHX`` 和 ``RPUSHX`` ，全都是左右对称的。

如果仅仅将 Redis 的列表结构用作单向列表，只在列表的其中一边对元素进行处理的话，那么有没有 ``LPOPRPUSH`` 是无所谓的：因为你总可以将新元素添加到列表的最左边，然后用 ``RPOPLPUSH`` 来对列表进行处理 —— 常见的消息队列和事件处理在 Redis 中通常都是这样实现的。

但是，缺少 ``LPOPRPUSH`` 在一些情况下也会造成不便，其中一个例子就是，没有 ``LPOPRPUSH`` ，你就没办法用很自然的方法实现循环列表，这种列表既可以向后旋转(``LPOPRPUSH list list`` ，并返回当前列表的尾节点)，也可以向前旋转(``RPOPLPUSH list list`` ，并返回列表的头节点)，以下是一个虚构的 ``LPOPRPUSH`` 例子：

::

    redis> LRANGE seq 0 -1
    1) "1"
    2) "2"
    3) "3"
    4) "4"

    redis> LPOPRPUSH seq seq
    "1"

    redis> LRANGE seq 0 -1
    1) "2"
    2) "3"
    3) "4"
    4) "1"

    redis> LPOPRPUSH seq seq
    "2"

    redis> LRANGE seq 0 -1
    1) "3"
    2) "4"
    3) "1"
    4) "2"

    redis> LPOPRPUSH seq seq
    "3"

    redis> LRANGE seq 0 -1
    1) "4"
    2) "1"
    3) "2"
    4) "3"


LPOPRPUSH 的简单实现
-----------------------------------------------

既然已经将 ``LPOPRPUSH`` 的前世今生都了解了一遍，现在，是时候亲自实现这个命令了。

可以确定的是， ``LPOPRPUSH`` 的调用方式应该和 ``RPOPLPUSH`` 一样，但是弹出元素和添加元素的位置正好相反，也就是说，执行命令 ``LPOPRPUSH source destination`` ，它应该原子性地完成以下三个操作：

1) 弹出 ``source`` 的表头元素(为了方便起见，我们将这个元素称为 ``item``)  
2) 将 ``item`` 添加到 ``destination`` 的最右边，成为 ``destination`` 列表的表尾元素  
3) 将 ``item`` 返回给客户端  

并且 ``LPOPRPUSH`` 的 ``source`` 和 ``destination`` 参数的值可以是同一个列表，从而制造一种(向后)旋转操作。

一个最简单的 ``LPOPRPUSH`` 实现可能是这样的：

.. code-block:: lua

    -- unsafe_lpoprpush.lua

    local redis = require 'redis'
    local client = redis.connect('127.0.0.1', 6379)

    function lpoprpush(source, destination)
        local item = client:lpop(source)
        if item ~= nil then
            client:rpush(destination, item)
            return item
        end
    end


使用 MULTI 、 EXEC 和 WATCH 实现安全的 LPOPRPUSH 函数
-----------------------------------------------------------

上一节的 ``LPOPRPUSH`` 实现在通常情况下可以达到我们想要的效果，但是它并不是一个原子性操作，没有提供安全性方面的保证。

举个例子，假如客户端在执行 ``client:lpop(source)`` 之后失败，那么被弹出的 ``item`` 元素就会白白丢失，无法被 ``RPUSH`` 命令推入到 ``destination`` 列表。

为了将 ``LPOPRPUSH`` 函数修改成一个原子性操作，我们不仅需要 ``MULTI`` 和 ``EXEC`` ，而且因为 ``LPOPRPUSH`` 是一个 CAS (check-and-set, 检查并修改)类型的操作，我们还需要用上 ``WATCH`` 来监视 ``source`` 和 ``destination`` 列表。

以下是新的 ``LPOPRPUSH`` 实现，它保证了 ``LPOP`` 和 ``RPUSH`` 两个操作的原子性，可以在任何情况下正确地执行工作：

.. code-block:: lua

    -- safe_lpoprpush.lua

    local redis = require 'redis'
    local client = redis.connect('127.0.0.1', 6379)

    function lpoprpush(source, destination)
        if source == destination then
            local watch = source    
        else
            local watch = {source, destination}
        end 

        local item = nil 
        local options = { watch = watch, cas = true, retry = 2 } 
        client:transaction(options, function(t)
            item = t:lpop(source)
            if item ~= nil then
                t:multi()
                t:rpush(destination, item)
            end
        end)
                                                                                                                       
        return item
    end


Redis 在 2.6 版本以前的问题
---------------------------------

将前面的第一版(不安全的) ``LPOPRPUSH`` 和第二版(安全的) ``LPOPRPUSH`` 放在一起进行对比是一个很有教益的练习，我们可以从中提炼出两个版本之间的一些共性和问题，比如说：

1) 两个版本使用的核心命令是完全一样的(不包括事务方面的命令)  
2) 因为第一版没办法保证原子性，所以就有了第二版  
3) 第二版使用 ``WATCH`` 、 ``MULTI`` 和 ``EXEC`` 保证了原子性，并成功将代码量上升了一倍！  

另一方面，当涉及到 ``WATCH`` 命令的使用时，又牵扯出了以下问题：

4) 多条命令在客户端和服务器之间跑来跑去，非常浪费带宽和时间  
5) ``WATCH`` 在业务高峰时期，会对吞吐量产生很大影响，因为失败的情况可能会发生得很频繁  
6) ``WATCH`` 的使用很容易出错(因为 Lua 的 ``transaction`` 函数可以指定要监视的键，所以出错的可能比较少，但是在另外一些语言，比如 Ruby 和 Python 中，如果不小心将 ``WATCH`` 放错了地方，就会引入很隐晦的竞争条件)  

需要注意的是，以上的这些问题并不是一个特例，它们是一大类 CAS 操作的共同难题，在一些(稍微)比较复杂的 Redis 模式中，这些问题并不罕见，究其原因，是因为在 2.6 版本以前， Redis 缺少一种自己的内嵌语言，因此即使像条件判断这样的简单操作，也要交由客户端去完成，而一旦命令需要在服务器和客户端两边来回处理的话，原子性又成了一个严峻的问题：在旧版 Redis 中， ``WATCH`` 和事务常常被用在很多不该用的地方，而初衷仅仅是为了保证原子性(上面的 ``LPOPRPUSH`` 实现就是一个很好的例子)。

幸运的是，随着 Redis 2.6 版本的出现，这种对 ``WATCH`` 和事务的滥用即将走向终点，因为在 Redis 2.6 版本中，新添加了对 Lua 脚本的支持，从而让我们可以将条件判断这类简单的操作和对 Redis 命令的调用都放到 Redis 服务器里完成，而这一切，仅仅需要一个 ``EVAL`` 命令。


EVAL
----------------

`EVAL <http://redis.readthedocs.org/en/latest/script/eval.html>`_ 是 Redis 2.6 版本新增命令的其中一个，同时也是最重要的一个，通过这个命令，可以直观、优雅且高效地解决像 ``LPOPRPUSH`` 这类 CAS 问题，文章稍后就会给出用脚本实现 ``LPOPRPUSH`` 的代码，但在此之前，不妨先来简单认识一下 ``EVAL`` 命令。

``EVAL`` 命令的调用形式是 ``EVAL script numkeys key [key ...] arg [arg ..]`` ，它的参数分别是：

``script`` ：一段 Lua 脚本代码(Lua 5.1版本)  
``numkeys`` ：用于指定键名参数(key name args)的个数  
``key [key ...]`` ：键名参数，在 Lua 中调用 Redis 命令时，那些被执行命令的键，可以在 Lua 脚本中通过全局数组 ``KEYS`` 来访问这些键名参数(数组下标以 ``1`` 为起始值)  
``arg [arg ...]`` ：附加参数，当在 Lua 中执行 Redis 命令时，用作命令的参数，可以在 Lua 脚本中通过全局数组 ``ARGV`` 来访问这些附加参数(数组下标以 ``1`` 为起始值)  

``script`` 参数和 ``numkeys`` 参数是必须的，通过给定一个脚本，并且将 ``numkeys`` 设为 ``0`` ，我们就可以用 ``EVAL`` 命令来执行简单的 Lua 求值了：

先用 ``EVAL`` 来个传统问候吧：

::

    redis> EVAL "return 'hello world'" 0
    "hello world"

然后做做初等数学计算：

::

    redis> EVAL "return 1 + 1" 0
    (integer) 2

    redis> EVAL "return 10/2" 0
    (integer) 5

又或者，来点儿条件判断式：

::

    redis> EVAL "if 1 == 1 then return 'good' else return 'bad' end" 0
    "good"

当然， ``EVAL`` 的真正威力不仅仅是写写 Lua 表达式那么简单，更关键的是，你可以使用 ``redis.call`` 函数，在 Lua 环境中执行 Redis 命令。

以下是两个 ``SET`` 和 ``GET`` 的例子：

::

    redis> EVAL "return redis.call('set', KEYS[1], ARGV[1])" 1 message "hello, moto"
    OK

    redis> EVAL "return redis.call('get', KEYS[1])" 1 message
    "hello, moto"

这两个脚本和以下的 Redis 命令等价：

::

    redis> SET message "hello, moto"
    OK

    redis> GET message
    "hello, moto"

以下是另外一个集合的例子：

::

    redis> EVAL "return redis.call('sadd', KEYS[1], ARGV[1])" 1 animal snake
    (integer) 1

    redis> EVAL "return redis.call('sadd', KEYS[1], unpack(ARGV))" 1 animal wolf lion tiger
    (integer) 3

它和以下这两个命令等价：

::

    redis> SADD animal snake
    (integer) 1

    redis> SADD animal wolf lion tiger
    (integer) 3

键名参数并不一定总是只能有一个，比如说，在执行 [RENAME](http://redis.readthedocs.org/en/latest/key/rename.html) 命令的时候，就需要两个键名参数：

::

    redis> EVAL "return redis.call('rename', KEYS[1], KEYS[2])" 2 old_name new_name
    OK

这个脚本和以下命令等价：

::

    redis> rename old_name new_name
    OK

关于 ``EVAL`` 命令的使用，最后要提醒的一点是，应该总是通过 ``KEYS`` 和 ``ARGV`` 两个变量来访问相应的键和参数，不要将键名和命令参数硬写到脚本里面，这会导致脚本无法使用 ``EVALSHA`` 命令来优化，也没有办法被 Redis 集群所执行。

这是一个典型的坏例子，不要这样做(尽管它可能暂时是可用的)：

::

    redis> EVAL "return redis.call('rpush', 'language', 'ruby', 'python', 'lua')" 0
    (integer) 3


LPOPRPUSH 的脚本实现
------------------------

在上一节，我们介绍了 ``EVAL`` 的两个特性：

1) 可以使用 ``EVAL`` 命令对 Lua 脚本进行求值  
2) 可以在 Lua 脚本里面使用 ``redis.call`` 执行 Redis 命令 

如果将这两个特性组合起来使用，就可以实现一些有趣的操作，比如说，我们可以写一个脚本，它只在键不存在的情况下对键执行 ``SET`` 命令，就像 Redis 的 ``SETNX`` 命令一样：

::

    redis> EVAL "if redis.call('exists', KEYS[1]) == 0 then return redis.call('set', KEYS[1], ARGV[1]) end" 1 date 2012.4.4
    OK

    redis> EVAL "if redis.call('exists', KEYS[1]) == 0 then return redis.call('set', KEYS[1], ARGV[1]) end" 1 date 2012.4.4
    (nil)

上面执行的脚本也是一个典型的 CAS 操作，它先检查一个键是否存在，然后根据反馈决定该怎么处理给定的键。如果在客户端执行 ``EVAL`` 里面的那一段代码，那可以肯定它是不是原子性操作，但是，在 ``EVAL`` 命令中，这个脚本并不会产生安全问题 —— 这是因为， ``EVAL`` 的执行是原子性的，这也是你需要知道的，关于 ``EVAL`` 的，第三个特性：

3) Redis 保证被 ``EVAL`` 命令所执行的脚本的原子性：当一个脚本正在运行的时候，不会有别的脚本或者别的 Redis 命令被执行。而当前所运行脚本的作用(effect)要么是不可见的(not visible)，要么就是已完成的(completed)。  

这样一来，在 Redis 2.6 版本以前一直困扰我们的很多 CAS 类型的问题，一下子就不复存在了：因为只要简单地将操作写成脚本，然后放到 ``EVAL`` 命令里运行，这样就再也不必担心原子性的问题了，也可以从此跟麻烦的 ``WATCH`` 命令说再见了。

作为一个演示 ``EVAL`` 真正威力的例子，我们回过头来，使用脚本实现之前讨论的 ``LPOPRPUSH`` 操作：

.. code-block:: lua

    -- script_lpoprpush.lua

    local redis = require 'redis'
    local client = redis.connect('127.0.0.1', 6379)

    function lpoprpush(source, destination)
        script = [[
            local item = redis.call('lpop', KEYS[1])
            if item ~= nil then
                redis.call('rpush', KEYS[2], item)
                return item
            end
        ]]

        return client:eval(script, 2, source, destination)
    end

我们可以将三个版本的 ``LPOPRPUSH`` 放在一起进行对比：

第一个版本是最简单的，但是它保证不了原子性。

第二个版本通过使用事务和 ``WATCH`` 保证了操作的原子性，但是也因此引入了很多不必要的复杂性。

第三个版本既保持了第一个版本的简单性(几乎就是客户端代码到 ``redis.call`` 函数的翻译)，在兼顾原子性的同时，又没有像第二版那样的不必要的复杂性，毫无疑问， ``LPOPRPUSH`` 的这个脚本实现是简单、高效而且优美的。

更重要的是，除了 ``LPOPRPUSH`` 之外，一大类 CAS 操作也可以用脚本的方式来解决，随着 Redis 2.6 版本的普及，可以预见的是，越来越多以前使用 ``WATCH`` 和事务来保证原子性的模式会逐渐被脚本实现所代替，更多有趣的新脚本模式也会陆续出现，这毫无疑问是非常让人期待的。


更多
------

关于 Redis 2.6 版本新增的脚本功能，这篇文章只是谈了其中的一小部分，很多有趣的特性因为篇幅关系都未能在文章里提及：比如使用 ``EVALSHA`` 优化带宽、使用 ``SCRIPT *`` 命令控制脚本缓存、脚本的沙箱和最大执行时间等等，关于这些特性，可以参考 Redis 的官方文档。


| huangz
| 2012.4.4
