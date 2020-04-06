Redis 处理和修复不合法 AOF 文件的方法
==========================================

当 Redis 运行在 AOF 持久化模式之下时，
每条对数据库进行修改的写命令都会被追加到 AOF 文件中。

举个例子，
如果当前 AOF 文件中的状态为：

::

    SET TODAY 2013-4-26

那么当用户执行命令 ``SET TOMORROW 2013-4-27`` 之后，
AOF 文件中的内容将变为：

::

    SET TODAY 2013-4-26

    SET TOMORROW 2013-4-27
    
Redis 服务器使用 ``O_APPEND`` 模式的 `open`_ 函数来打开 AOF 文件，
并通过 `write`_ 函数来将命令内容追加到 AOF 文件的末尾。

在一般情况下，
`write`_ 函数可以无错地、原子地完成 ——
除非出现以下两种情况的任意一种：

1. 写入错误：在写入的过程中，出现写入操作被打断，磁盘空间不足，或者磁盘出现物理故障等问题。

2. 停机故障：在写入的过程中，发生 Redis 服务器进程别强制杀死，或者服务器的宿主机器被强制停机等情况。

比如说，
在以下 AOF 文件中，
命令 ``SET TOMORROW 2013-4-27`` 就只有 ``SET`` 部分被写入到了 AOF 文件中：

::

    SET TODAY 2013-4-26

    SET
 
如果在 Redis 运行的过程中发生了写入错误，
那么 Redis 将停止服务器，
并打印一个错误来通知用户。

另一方面，
如果在启动 Redis 时，
用户试图将带有不完整数据的 AOF 文件提供给 Redis 时，
Redis 将会拒绝使用用户所提供的 AOF 文件来还原数据，
因为错误的 AOF 文件会破坏数据的一致性。

这篇文章将对 Redis 处理错误 AOF 文件的方法进行介绍，
并讲解 AOF 修复软件 ``redis-check-aof`` 的运作原理。


前提知识
--------------------

本文的讨论涉及 Redis 的命令、通讯协议、AOF 持久化和系统文件操作等几个方面的内容，
如果读者不具备这些前提知识的话，
请先阅读以下文档：

- `Redis 命令参考 <http://redis.readthedocs.org>`_

- `Redis 通讯协议 <http://redis.readthedocs.org/en/latest/topic/protocol.html>`_

- `Redis 持久化 <http://redis.readthedocs.org/en/latest/topic/persistence.html>`_

- `write 函数手册 <http://linux.die.net/man/2/write>`_ 、 `open 函数手册 <http://linux.die.net/man/2/open>`_ 、 `ftruncate 函数手册 <http://linux.die.net/man/2/ftruncate>`_ 和 `exit 函数手册 <http://linux.die.net/man/2/exit>`_


写入错误
--------------------

前面说到，
AOF 文件的写入可能遭遇两种问题，
一种是写入错误，
另一种是停机故障，
本节先来介绍 Redis 是如何处理写入错误的。

首先要说明的是，
写入错误并非致命错误，
所以当出现写入错误的时候，
Redis 仍然处于运行当中，
因此，
Redis 可以根据写入错误的情况，
尝试对错误进行修复。

以下是 Redis 源码 ``aof.c`` 文件中的其中一行代码，
它描述了 Redis 是如何将要追加的新内容写入到 AOF 文件里的：

.. code-block:: c

    nwritten = write(server.aof_fd,server.aof_buf,sdslen(server.aof_buf));

其中 ``server.aof_fd`` 是 AOF 文件的描述符，
``server.aof_buf`` 是要被写入到 AOF 文件的内容，
而 ``sdslen(server.aof_buf)`` 则是被写入内容的长度。

取决于 ``nwritten`` 的值（也即是 `write`_ 函数的返回值），
有以下四种情况出现：

1. ``nwritten == sdslen(server.aof_buf)`` ：内容全部被成功写入到 AOF 文件中。

2. ``nwritten == -1`` ：写入出错，没有任何内容被写入。

3. ``nwritten == 0`` ，并且 ``errno`` 被设置：和情况 2 一样。

4. ``0 <= nwritten < sdslen(server.aof_buf)`` ，
   并且当 ``nwritten == 0`` 时，
   ``errno`` 没有被设置：
   内容没有被全部写入。

在情况 1 中，写入没有发生任何错误，程序继续运行。

在情况 2 和 3 中，
写入可能发生了比较严重的错误，
最坏情况下，
也许已经没办法再读写硬盘了：
这种情况单靠 Redis 是不能处理的，
而且出了这种问题，
Redis 也不指望自己能再继续执行下去了，
所以 Redis 会打印一个错误日志，
然后直接调用 `exit`_ 退出。

情况 4 的处理方式最有趣，
因为在这种情况下，
磁盘有可能可以继续进行读写 ——
因此，
程序会使用以下代码，
尝试自动修复出错的 AOF 文件：

::

    // 尝试移除新追加的不完整内容
    if (ftruncate(server.aof_fd, server.aof_current_size) == -1) {
        redisLog(REDIS_WARNING, "Could not remove short write "
                                "from the append-only file.  Redis may refuse "
                                "to load the AOF the next time it starts.  "
                                "ftruncate: %s", strerror(errno));
    }    

所谓的自动修复操作，
实际上就是尝试将 `write`_ 写入的不完整内容从 AOF 文件中删除掉：
在上面的代码中，
``server.aof_current_size`` 记录了 AOF 文件在执行 `write`_ 之前的大小，
通过 `ftruncate`_ 函数，
可以将 `write`_ 写入的不完整内容删除掉，
从而将 AOF 文件还原到执行 `write`_ 之前的状态。

比如说，
如果这是 AOF 文件执行 `write`_ 之前的内容：

::

    SET TODAY 2013-4-26
 
当 ``write`` 出错之后，文件的内容变为：

::

    SET TODAY 2013-4-26

    SET

如果 ``ftruncate`` 函数能够成功执行，
那么 AOF 文件的内容将会变回执行 ``write`` 之前的样子：

::

    SET TODAY 2013-4-26

无论自动修复是否成功，
程序都会调用 `exit`_ 退出，
用户必须在这之后手动重启 Redis 服务器。

自动修复能否成功取决于 `ftruncate`_ 函数能否成功执行，
而 `ftruncate`_ 能否成功执行取决于磁盘能否被正常读写：

- 如果自动修复成功的话，
  那么客户在服务器退出之后，
  可以无须执行任何额外动作，
  直接重启服务器。

- 另一方面，
  如果自动修复失败，
  那么客户在重启服务器之前，
  必须使用 ``redis-check-aof`` 程序手动修复 AOF 文件，
  否则的话，
  服务器将拒绝重新启动。


停机故障
----------------

前面说到，
在碰到无法自动修复的写入错误时，
服务器会留下一个带有不完整数据的 AOF 文件，
然后退出。

停机故障和无法修复的写入错误的情况一样：
因为 Redis 进程被强制终结，
所以 AOF 程序在执行 `write`_ 的中途会被打断，
并且留下一个写入了不完整数据的 AOF 文件。

在用户启动 Redis 服务器，
并且服务器试图载入 AOF 文件的数据之前，
程序会先对 AOF 文件的完整性进行检查，
如果 AOF 文件带有不完整的写入数据，
那么为了保护数据的一致性，
Redis 会拒绝载入这个 AOF 文件，
并打印以下日志信息：

::

    $ redis-server 
    [6842] 26 Apr 10:43:48.842 # Server started, Redis version 2.9.9
    [6842] 26 Apr 10:43:48.842 # Unexpected end of file reading the append only file

Redis 附带的 ``redis-check-aof`` 可以解决这类因为 AOF 出错而无法启动的问题：
它可以移除 AOF 文件所包含的不完整的数据，
使得 Redis 可以重新载入 AOF 文件中的数据。

举个例子，
以下是一个不完整的 AOF 文件，
程序本来打算将命令 ``SET TOMORROW 2013-4-27`` 写入到 AOF 文件里，
但没想到只写完 ``*3\r\n$3\r\nSET\r\n`` 就遇上了停机故障
（开头的 :ref:`SELECT <redisref:select>` 命令是 AOF 程序自动加上的）：

::

    *2
    $6
    SELECT
    $1
    0
    *3
    $3
    SET
    $5
    TODAY
    $9
    2013-4-26
    *3
    $3
    SET   

执行 ``redis-check-aof`` 程序可以检查给定的 AOF 文件是否合法（有没有包含不完整的数据）。

举个例子，
我们可以用 ``redis-check-aof`` 来检查上面展示的 AOF 文件：

::

    $ redis-check-aof invalid.aof 
    AOF analyzed: size=75, ok_up_to=62, diff=13
    AOF is not valid

程序显示，
``invalid.aof`` 是一个不合法的 AOF 文件：
它的前 ``62`` 字节是正常的，
但是之后的 ``13`` 字节是错误的。

使用命令 ``redis-check-aof --fix`` 可以修复一个不合法的 AOF 文件：

::

    $ redis-check-aof --fix invalid.aof 
    AOF analyzed: size=75, ok_up_to=62, diff=13
    This will shrink the AOF from 75 bytes, with 13 bytes, to 62 bytes
    Continue? [y/N]: y
    Successfully truncated AOF

程序会定位到出错的地方，
并询问用户，
是否要对文件进行截断，
如果选择 ``y`` 的话，
就会对指定的 AOF 文件进行修复。

以下是修复后的 AOF 文件：

::

    *2
    $6
    SELECT
    $1
    0
    *3
    $3
    SET
    $5
    TODAY
    $9
    2013-4-26

可以看到，出错的原因 —— 那个不完整的 :ref:`SET <redisref:set>` 命令已经被删除了。

现在再次使用 ``redis-check-aof`` 程序检查这个 AOF 文件的话，
程序会显示这是一个合法的 AOF 文件：

::

    $ redis-check-aof invalid.aof 
    AOF analyzed: size=62, ok_up_to=62, diff=0
    AOF is valid

并且 Redis 也可以正确地载入这个 AOF 文件：

::

    $ redis-server
    [12948] 26 Apr 18:40:43.355 # Server started, Redis version 2.9.9
    [12948] 26 Apr 18:40:43.355 * DB loaded from append only file: 0.000 seconds
    [12948] 26 Apr 18:40:43.355 * The server is now ready to accept connections on port 6379
    

修复命令错误
--------------------------------------

在上一个小节，
我们看到了如何使用 ``redis-check-aof`` 来检查一个 AOF 文件是否合法，
并且知道了怎样使用 ``redis-check-aof --fix`` 来修复一个 AOF 文件，
在这一节，
我们就来看看，
``redis-check-aof`` 程序是如何定位错误的命令，
以及它又是怎样删除不完整的命令内容的。

还是以前面的那个不合法的 AOF 文件作为例子：

::

    *2
    $6
    SELECT
    $1
    0
    *3
    $3
    SET
    $5
    TODAY
    $9
    2013-4-26
    *3
    $3
    SET   

以下是 ``redis-check-aof`` 处理上面这个 AOF 文件时的步骤：

1. 读入 ``*2\r\n`` ，确认到接下来的是一个两个参数的命令（一个是命令、一个是参数）：

   a. 读入 ``$6\r\n`` ，并确认接下来的参数为 ``6`` 字节长的 ``SELECT`` 。

   b. 读入 ``$1\r\n`` ，并确认接下来的参数为 ``1`` 字节长的 ``0`` 。

2. 读入 ``*3\r\n`` ，确认到接下来的是一个三个参数的命令（一个是命令，两个是参数）：

   a. 读入 ``$3\r\n`` ，并确认 ``3`` 字节长的参数 ``SET`` 。

   b. 读入 ``$5\r\n`` ，并确认 ``5`` 字节长的参数 ``TODAY`` 。

   c. 读入 ``$9\r\n`` ，并确认 ``9`` 字节长的参数 ``2013-4-26`` 。

3. 读入 ``*3\r\n`` ，确认到接下来的是一个三个参数的命令（一个是命令，两个是参数）：

   a. 读入 ``$3\r\n`` ，并确认 ``3`` 字节长的参数 ``SET`` 。

   b. 程序试图读入三个参数中的第二个，但是意外地发现读入已经到达了文件的末尾，出错！

上面是从协议的角度来分析 AOF 中的内容，
如果我们从文本的角度来看，
分析的步骤就会简单得多：

::

    SELECT 0

    SET TODAY 2013-4-26

    SET

1. 程序首先读入 ``SELECT`` 命令，还有它的参数 ``0`` 。

2. 程序首先读入 ``SET`` 命令，接着是它的第二个参数 ``TODAY`` ，然后是它的第三个参数 ``2013-4-26`` 。

3. 程序首先读入 ``SET`` 命令，然后试图读取第二个参数，但是失败，于是程序报告错误。

这就是 ``redis-check-aof`` 程序的运作方式 ——
按 Redis 通讯协议的格式，
按顺序读入 AOF 文件中的每个命令的每个参数，
并对这些参数进行检查：

1. 如果遇到了不符合协议格式的内容，或者命令在不该结束的地方结束了，那么这个 AOF 文件就是不合法的。

2. 如果没有发现参数错误，并且顺利地读到了文件末尾，那么这个 AOF 文件就是合法的。

每次读入一个命令之前，
``redis-check-aof`` 都会记录命令开始的偏移量，
当某个命令出现错误时：

- 如果执行的是 ``redis-check-aof`` ，那么程序就向用户报告 AOF 文件的错误。

- 如果执行的是 ``redis-check-aof --fix`` ，那么程序就询问用户，要不要对 AOF 文件进行修复。

以下是 ``redis-check-aof`` 检查 ``invalid.aof`` 的结果：

::

    $ redis-check-aof invalid.aof 
    AOF analyzed: size=75, ok_up_to=62, diff=13
    AOF is not valid

程序显示，
``invalid.aof`` 文件总共 ``75`` 字节，
文件的前 ``62`` 字节是正常的，
而后面的 ``13`` 字节则是写入不完整的数据。

使用 ``od -c`` 命令，
可以正好看到，
``62`` 字节之后，
就是出错的 :ref:`SET <redisref:set>` 命令的协议内容：

::

    $ od -c invalid.aof 
    0000000   *   2  \r  \n   $   6  \r  \n   S   E   L   E   C   T  \r  \n
    0000020   $   1  \r  \n   0  \r  \n   *   3  \r  \n   $   3  \r  \n   S
    0000040   E   T  \r  \n   $   5  \r  \n   T   O   D   A   Y  \r  \n   $
    0000060   9  \r  \n   2   0   1   3   -   4   -   2   6  \r  \n   *   3
    0000100  \r  \n   $   3  \r  \n   S   E   T  \r  \n

修复 AOF 文件的办法就是使用 `ftruncate`_ 将出错命令整个删除掉。

以下命令展示了 ``invalid.aof`` 在被修复之后的样子：

::

    $ redis-check-aof --fix invalid.aof 
    AOF analyzed: size=75, ok_up_to=62, diff=13
    This will shrink the AOF from 75 bytes, with 13 bytes, to 62 bytes
    Continue? [y/N]: y
    Successfully truncated AOF

    $ od -c invalid.aof 
    0000000   *   2  \r  \n   $   6  \r  \n   S   E   L   E   C   T  \r  \n
    0000020   $   1  \r  \n   0  \r  \n   *   3  \r  \n   $   3  \r  \n   S
    0000040   E   T  \r  \n   $   5  \r  \n   T   O   D   A   Y  \r  \n   $
    0000060   9  \r  \n   2   0   1   3   -   4   -   2   6  \r  \n
    0000076

可以看到，
不合法的 :ref:`SET <redisref:set>` 命令已经被删掉了，
修复后的文件只保留了合法的前 ``62`` 字节的内容。


修复事务错误
-------------------

在上一节，
我们看到了 ``redis-check-aof`` 如何分析、定位并删除出错的命令内容，
从而修复一个 AOF 文件。

但是在一些情况下，
单单删除一个命令并不能达到修复 AOF 文件的要求，
考虑以下这个 AOF 文件：

::

    *2
    $6
    SELECT
    $1
    0
    *3
    $3
    SET
    $5
    TODAY
    $9
    2013-4-26
    *1
    $5
    MULTI
    *3
    $3
    SET

在 AOF 文件的最后，
程序只写入了一个不完整的 :ref:`SET <redisref:set>` 命令，
但是，
就算我们将这个 :ref:`SET <redisref:set>` 命令删除，
这个 AOF 文件也仍然不合法：
因为在 :ref:`SET <redisref:set>` 之前的 :ref:`MULTI <redisref:multi>` 命令，
也同样是不合法的 —— 
这个 :ref:`MULTI <redisref:multi>` 没有和它相对应的 :ref:`EXEC <redisref:exec>` 命令。

事实上，
即使遇上这种情况，
``redis-check-aof`` 也有办法修复：

::

    $ redis-check-aof invalid_transaction.aof 
    0x              5a: Reached EOF before reading EXEC for MULTI
    AOF analyzed: size=90, ok_up_to=62, diff=28
    AOF is not valid

    $ od -c invalid_transaction.aof 
    0000000   *   2  \r  \n   $   6  \r  \n   S   E   L   E   C   T  \r  \n
    0000020   $   1  \r  \n   0  \r  \n   *   3  \r  \n   $   3  \r  \n   S
    0000040   E   T  \r  \n   $   5  \r  \n   T   O   D   A   Y  \r  \n   $
    0000060   9  \r  \n   2   0   1   3   -   4   -   2   6  \r  \n   *   1
    0000100  \r  \n   $   5  \r  \n   M   U   L   T   I  \r  \n   *   3  \r
    0000120  \n   $   3  \r  \n   S   E   T  \r  \n
    0000132

    $ redis-check-aof --fix invalid_transaction.aof 
    0x              5a: Reached EOF before reading EXEC for MULTI
    AOF analyzed: size=90, ok_up_to=62, diff=28
    This will shrink the AOF from 90 bytes, with 28 bytes, to 62 bytes
    Continue? [y/N]: y
    Successfully truncated AOF

    $ od -c invalid_transaction.aof 
    0000000   *   2  \r  \n   $   6  \r  \n   S   E   L   E   C   T  \r  \n
    0000020   $   1  \r  \n   0  \r  \n   *   3  \r  \n   $   3  \r  \n   S
    0000040   E   T  \r  \n   $   5  \r  \n   T   O   D   A   Y  \r  \n   $
    0000060   9  \r  \n   2   0   1   3   -   4   -   2   6  \r  \n
    0000076

以下是被修复后的 AOF 文件：

::

    *2
    $6
    SELECT
    $1
    0
    *3
    $3
    SET
    $5
    TODAY
    $9
    2013-4-26

可以看到，
不止是 :ref:`SET <redisref:set>` 命令，
``redis-check-aof`` 程序将整个事务，
包括 :ref:`MULTI <redisref:multi>` 和 :ref:`SET <redisref:set>` 在内的两个命令都删除了。

这就是 ``redis-check-aof`` 处理事务的方式：
当它在读入 AOF 文件时，
如果遇到 :ref:`MULTI` 命令，
那么它就记录 :ref:`MULTI <redisref:multi>` 命令开始时的偏移量：

1. 如果在事务的内部，
   有任何一个命令出错了，
   那么程序将删除 :ref:`MULTI` 命令，以及这个命令之后的所有文件内容。

2. 如果程序能够成功找到和 :ref:`MULTI <redisref:multi>` 相匹配的 :ref:`EXEC <redisref:exec>` ，
   那么程序就将偏移量移动到 :ref:`EXEC <redisref:exec>` 的末尾，
   也就是下个命令开始的地方，
   并继续进行读入分析，
   一直到整个 AOF 文件读入完毕为止。

回过头来观察被修复之前的 ``invalid_transaction.aof`` 文件，
``redis-check-aof`` 将 :ref:`MULTI <redisref:multi>` 命令，
以及命令之后的共 ``28`` 个字节都标记为非法数据，
而不仅仅是标记 :ref:`SET <redisref:set>` 那么简单：

::

    $ redis-check-aof invalid_transaction.aof
    0x              5a: Reached EOF before reading EXEC for MULTI
    AOF analyzed: size=90, ok_up_to=62, diff=28
    AOF is not valid

    $ od -c invalid_transaction.aof
    0000000   *   2  \r  \n   $   6  \r  \n   S   E   L   E   C   T  \r  \n
    0000020   $   1  \r  \n   0  \r  \n   *   3  \r  \n   $   3  \r  \n   S
    0000040   E   T  \r  \n   $   5  \r  \n   T   O   D   A   Y  \r  \n   $
    0000060   9  \r  \n   2   0   1   3   -   4   -   2   6  \r  \n   *   1
    0000100  \r  \n   $   5  \r  \n   M   U   L   T   I  \r  \n   *   3  \r
    0000120  \n   $   3  \r  \n   S   E   T  \r  \n
    0000132


小结
--------

以上就是本文的所有内容了。

我们首先观察了 Redis 是如何在磁盘出错时进行自动修复的，
然后学习了使用 ``redis-check-aof`` 对 AOF 文件进行手动修复的步骤，
并研究了 ``redis-check-aof`` 修复一般命令、以及修复事务命令时的工作原理。

学习这些知识可以让我们更好地应对 AOF 出错之类的问题，
还可以加深我们对 AOF 程序、以及 Redis 通讯协议等方面的熟悉程度，
如果有兴趣进一步地了解 ``redis-check-aof`` 程序的话，
可以阅读这份 `带注释的 redis-check-aof.c 源码 <https://github.com/huangz1990/annotated_redis_source/blob/unstable/src/redis-check-aof.c>`_ 。

| huangz
| 2013.4.26

.. _open: http://linux.die.net/man/2/open

.. _write: http://linux.die.net/man/2/write

.. _exit: http://linux.die.net/man/2/exit

.. _ftruncate: http://linux.die.net/man/2/ftruncate
