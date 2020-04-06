Redis 发布与订阅模式匹配符示例
=========================================

一直以来，
我对 Redis 的订阅与发布功能都有一个误解：
因为 Redis 在文档中总是使用带 ``*`` 符号的模式来做示例，
我一直错误地认为 Redis 的模式只能使用 ``*`` 符号来做匹配符。

直到前几天，
因为翻译 Redis `订阅与发布功能文档 <http://redis.readthedocs.org/en/latest/topic/pubsub.html>`_ 的缘故，
我看了一下负责对模式进行匹配的 ``stringmatchlen`` 函数的 `代码 <https://github.com/antirez/redis/blob/unstable/src/util.c#L43>`_ ，
这时才发现，
原来 Redis 的模式名中不仅可以包含 ``*`` 符号，
还可以使用其他 ``glob`` 风格的匹配符，
比如 ``?`` 和 ``[...]`` 。

.. tip::

    ``glob`` 程序的文档可以通过 ``man 7 glob`` 命令查看，
    其中的 Wildcard Matching 小节记录了各个匹配符的用法。

因为 Redis 的文档里并没有对除 ``*`` 符号之外的其他匹配符进行介绍，
所以我想如果有一篇专门介绍 Redis 发布与订阅模式的文章应该也会对其他人有帮助，
这就是本文的写作目的。

在接下来的小节里，
每个小节将介绍一种 Redis 模式可以使用的匹配符。



``*`` 匹配符
------------------

``*`` 匹配符可以匹配任意的字符串，
包括空字符串。

比如说，
模式 ``*news`` 可以匹配 ``sport news`` 、 ``breaking news`` ：

::

    redis 127.0.0.1:6379> PSUBSCRIBE *news
    Reading messages... (press Ctrl-C to quit)

    1) "psubscribe"
    2) "*news"
    3) (integer) 1

    1) "pmessage"
    2) "*news"
    3) "sport news"
    4) "hello world from sport news"

    1) "pmessage"
    2) "*news"
    3) "breaking news"
    4) "hello world from breaking news"

或者正好是字符串 ``news`` ：

::

    1) "pmessage"
    2) "*news"
    3) "news"
    4) "hello world"

等等。


``?`` 匹配符
-------------------

``?`` 匹配符可以匹配任意一个字符。

比如说，
``Re?is`` 可以匹配 ``Redis`` 、 ``ReDis`` 、 ``Regis`` 、 ``Re-is`` 等等。

以下是一个订阅 ``Re?is`` 模式的客户端：

::

    redis 127.0.0.1:6379> PSUBSCRIBE Re?is
    Reading messages... (press Ctrl-C to quit)

    1) "psubscribe"
    2) "Re?is"
    3) (integer) 1

当有信息发送到 ``Redis`` 频道的时候，
它将接收到以下信息：

::


    1) "pmessage"
    2) "Re?is"
    3) "Redis"
    4) "hello from channel Redis"

另外，
因为 ``ReDis`` 和 ``Re-is`` 等频道也和 ``Re?is`` 模式匹配，
所以当有信息发送到这些频道的时候，
订阅 ``Re?is`` 模式的客户端也会收到信息：

::

    1) "pmessage"
    2) "Re?is"
    3) "ReDis"
    4) "hello from channel ReDis"

    1) "pmessage"
    2) "Re?is"
    3) "Re-is"
    4) "hello from channel Re-is"


``[...]`` 匹配符
---------------------------

``[...]`` 匹配符可以匹配给定范围内的单个字符，
其中，
``[`` 和 ``]`` 包围的内容指定了这个字符可能的值。

比如说，
如果我们想匹配 ``Redis`` 、 ``Tedis`` 和 ``Uedis`` 三个频道的信息，
那么可以订阅模式 ``[RTU]edis`` ：

::

    redis 127.0.0.1:6379> PSUBSCRIBE [RTU]edis
    Reading messages... (press Ctrl-C to quit)

    1) "psubscribe"
    2) "[RTU]edis"
    3) (integer) 1

当有信息发送给 ``Redis`` 、 ``Tedis`` 或 ``Uedis`` 中的任意一个时，
订阅客户端都会收到信息：

::

    1) "pmessage"
    2) "[RTU]edis"
    3) "Redis"
    4) "hello from Redis"

    1) "pmessage"
    2) "[RTU]edis"
    3) "Tedis"
    4) "hello from Tedis"

    1) "pmessage"
    2) "[RTU]edis"
    3) "Uedis"
    4) "hello from Uedis"

和 ``glob`` 程序一样，
Redis 的 ``[...]`` 匹配符里面也可以使用 ``-`` 符号（横杠）来指定范围。

比如说，
要匹配频道 ``iPhone4`` 、 ``iPhone5`` 和 ``iPhone6`` ，
可以订阅模式 ``iPhone[4-6]`` ：

::

    redis 127.0.0.1:6379> PSUBSCRIBE iPhone[4-6]
    Reading messages... (press Ctrl-C to quit)
    1) "psubscribe"
    2) "iPhone[4-6]"
    3) (integer) 1

    1) "pmessage"
    2) "iPhone[4-6]"
    3) "iPhone4"
    4) "..."

    1) "pmessage"
    2) "iPhone[4-6]"
    3) "iPhone5"
    4) "..."

    1) "pmessage"
    2) "iPhone[4-6]"
    3) "iPhone6"
    4) "..."

另外，
要匹配 ``Aedis`` 、 ``Bedis`` 、 ``Cedis`` 一直到 ``Zedis`` 频道，
可以订阅 ``[A-Z]edis`` 模式：

::

    redis 127.0.0.1:6379> PSUBSCRIBE [A-Z]edis
    Reading messages... (press Ctrl-C to quit)
    1) "psubscribe"
    2) "[A-Z]edis"
    3) (integer) 1

    1) "pmessage"
    2) "[A-Z]edis"
    3) "Aedis"
    4) "..."

    1) "pmessage"
    2) "[A-Z]edis"
    3) "Redis"
    4) "..."

    1) "pmessage"
    2) "[A-Z]edis"
    3) "Zedis"
    4) "..."

.. note::

    因为 Redis 的 :ref:`SUBSCRIBE 命令<redisref:subscribe>` 可以同时订阅多个频道，
    所以实际上使用 ``-`` 符号来指定范围并不是十分必要的。

    比如说，
    命令 ``PSUBSCRIBE iPhone[4-6]`` 和命令 ``SUBSCRIBE iPhone4 iPhone5 iPhone6`` 的效果是一样的，
    它们之间的唯一不同之处是前者订阅的是模式，
    而后者订阅的是频道。

    从目前的频道和模式的实现来看，
    使用频道的效率要比模式高（具体请参考《Redis 设计与实现》的《\ `发布与订阅 <http://www.redisbook.com/en/latest/feature/pubsub.html>`_\ 》章节），
    因此，
    如果可以的话，
    应该尽量使用频道而不是模式。


任意组合多个匹配符
----------------------

上面介绍了 ``*`` 、 ``?`` 和 ``[...]`` 三个匹配符，
要进一步扩大模式的匹配能力，
可以组合使用多个匹配符：
Redis 允许一个模式中包含多个匹配符，
并且每个匹配符的数量没有限制。

比如说，
模式 ``[ig]Phone*`` 可以匹配 ``iPhone`` 、 ``gPhone`` 、 ``iPhone news`` 、 ``gPhone 7`` 等任意频道：

::

    redis 127.0.0.1:6379> PSUBSCRIBE [ig]Phone*
    Reading messages... (press Ctrl-C to quit)
    1) "psubscribe"
    2) "[ig]Phone*"
    3) (integer) 1

    1) "pmessage"
    2) "[ig]Phone*"
    3) "iPhone"
    4) "..."

    1) "pmessage"
    2) "[ig]Phone*"
    3) "gPhone"
    4) "..."

    1) "pmessage"
    2) "[ig]Phone*"
    3) "iPhone news"
    4) "..."

    1) "pmessage"
    2) "[ig]Phone*"
    3) "gPhone 7"
    4) "..."

而模式 ``*news*`` 则可以匹配任何包含 ``news`` 这一字符串的频道，
比如 ``news`` 、 ``release news of Redis`` 、 ``lastes news`` ，
诸如此类：

::

    redis 127.0.0.1:6379> PSUBSCRIBE *news*
    Reading messages... (press Ctrl-C to quit)
    1) "psubscribe"
    2) "*news*"
    3) (integer) 1

    1) "pmessage"
    2) "*news*"
    3) "news"
    4) "..."

    1) "pmessage"
    2) "*news*"
    3) "release news of Redis"
    4) "..."

    1) "pmessage"
    2) "*news*"
    3) "lastes news"
    4) "..."


总结
-------

以上就是本文的全部内容了。

本文对 Redis :ref:`PSUBSCRIBE 命令<redisref:psubscribe>` 支持的各个匹配符进行了简单的介绍，
希望各位可以从这些简单的例子中获得启发，
创作出更多更强大的模式，
并将它们应用到程序中去。

| huangz
| 2013.5.13
