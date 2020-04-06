Redis 4.0 新功能简介
============================

Redis 的作者 antirez 在三天之前通过博客文章《\ `The first release candidate of Redis 4.0 is out <http://antirez.com/news/110>`_\ 》发布了 Redis 4.0 的第一个 RC 版本，
在博文中他说，
因为这个新版本的 Redis 出现了多项改变，
所以他决定从原来的 3.x 版本直接跳到 4.0 版本，
以此来强调这次更新的变化之大。

本文将对 Redis 4.0 的各项主要新功能做一个简单的介绍，
方便大家第一时间了解到这次更新的主要内容。



模块系统
-----------

Redis 4.0 发生的最大变化就是加入了模块系统，
这个系统可以让用户通过自己编写的代码来扩展和实现 Redis 本身并不具备的功能，
具体使用方法可以参考 antirez 的博文《Redis Loadable Module System》：
http://antirez.com/news/106

因为模块系统是通过高层次 API 实现的，
它与 Redis 内核本身完全分离、互不干扰，
所以用户可以在有需要的情况下才启用这个功能，
以下是 ``redis.conf`` 中记载的模块载入方法：

::

    ################################## MODULES #####################################

    # Load modules at startup. If the server is not able to load modules
    # it will abort. It is possible to use multiple loadmodule directives.
    #
    # loadmodule /path/to/my_module.so
    # loadmodule /path/to/other_module.so

目前已经有人使用这个功能开发了各种各样的模块，
比如 Redis Labs 开发的一些模块就可以在 http://redismodules.com 看到，
此外 antirez 自己也使用这个功能开发了一个神经网络模块：
https://github.com/antirez/neural-redis 

模块功能使得用户可以将 Redis 用作基础设施，
并在上面构建更多功能，
这给 Redis 带来了无数新的可能性。



PSYNC 2.0
-------------

新版本的 ``PSYNC`` 命令解决了旧版本的 Redis 在复制时的一些不够优化的地方：

- 在旧版本 Redis 中，
  如果一个从服务器在 FAILOVER 之后成为了新的主节点，
  那么其他从节点在复制这个新主的时候就必须进行全量复制。
  在 Redis 4.0 中，
  新主和从服务器在处理这种情况时，
  将在条件允许的情况下使用部分复制。

- 在旧版本 Redis 中，
  一个从服务器如果重启了，
  那么它就必须与主服务器重新进行全量复制，
  在 Redis 4.0 中，
  只要条件允许，
  主从在处理这种情况时将使用部分复制。



缓存驱逐策略优化
-------------------

新添加了 Last Frequently Used 缓存驱逐策略，
具体信息见 antirez 的博文《Random notes on improving the Redis LRU algorithm》： http://antirez.com/news/109

另外 Redis 4.0 还对已有的缓存驱逐策略进行了优化，
使得它们能够更健壮、高效、快速和精确。



非阻塞 DEL 、 FLUSHDB 和 FLUSHALL
---------------------------------------

在 Redis 4.0 之前，
用户在使用 ``DEL`` 命令删除体积较大的键，
又或者在使用 ``FLUSHDB`` 和 ``FLUSHALL`` 删除包含大量键的数据库时，
都可能会造成服务器阻塞。

为了解决以上问题，
Redis 4.0 新添加了 ``UNLINK`` 命令，
这个命令是 ``DEL`` 命令的异步版本，
它可以将删除指定键的操作放在后台线程里面执行，
从而尽可能地避免服务器阻塞：

::

    redis> UNLINK fruits
    (integer) 1

因为一些历史原因，
执行同步删除操作的 ``DEL`` 命令将会继续保留。

此外，
Redis 4.0 中的 ``FLUSHDB`` 和 ``FLUSHALL`` 这两个命令都新添加了 ``ASYNC`` 选项，
带有这个选项的数据库删除操作将在后台线程进行：

::

    redis> FLUSHDB ASYNC
    OK

    redis> FLUSHALL ASYNC
    OK



交换数据库
--------------

Redis 4.0 对数据库命令的另外一个修改是新增了 ``SWAPDB`` 命令，
这个命令可以对指定的两个数据库进行互换：
比如说，
通过执行命令 ``SWAPDB 0 1`` ，
我们可以将原来的数据库 0 变成数据库 1 ，
而原来的数据库 1 则变成数据库 0 。

以下是一个使用 ``SWAPDB`` 的例子：

::

    redis> SET your_name "huangz"  -- 在数据库 0 中设置一个键
    OK

    redis> GET your_name
    "huangz"

    redis> SWAPDB 0 1  -- 互换数据库 0 和数据库 1
    OK

    redis> GET your_name  -- 现在的数据库 0 已经没有之前设置的键了
    (nil)

    redis> SELECT 1  -- 切换到数据库 1 
    OK

    redis[1]> GET your_name  -- 之前在数据库 0 设置的键现在可以在数据库 1 找到
    "huangz"                 -- 证明两个数据库已经互换



混合 RDB-AOF 持久化格式
-----------------------------

Redis 4.0 新增了 RDB-AOF 混合持久化格式，
这是一个可选的功能，
在开启了这个功能之后，
AOF 重写产生的文件将同时包含 RDB 格式的内容和 AOF 格式的内容，
其中 RDB 格式的内容用于记录已有的数据，
而 AOF 格式的内存则用于记录最近发生了变化的数据，
这样 Redis 就可以同时兼有 RDB 持久化和 AOF 持久化的优点 ——
既能够快速地生成重写文件，
也能够在出现问题时，
快速地载入数据。

这个功能可以通过 ``aof-use-rdb-preamble`` 选项进行开启，
``redis.conf`` 文件中记录了这个选项的使用方法：

::

    # When rewriting the AOF file, Redis is able to use an RDB preamble in the
    # AOF file for faster rewrites and recoveries. When this option is turned
    # on the rewritten AOF file is composed of two different stanzas:
    #
    #   [RDB file][AOF tail]
    #
    # When loading Redis recognizes that the AOF file starts with the "REDIS"
    # string and loads the prefixed RDB file, and continues loading the AOF
    # tail.
    #
    # This is currently turned off by default in order to avoid the surprise
    # of a format change, but will at some point be used as the default.
    aof-use-rdb-preamble no



内存命令
------------

新添加了一个 ``MEMORY`` 命令，
这个命令可以用于视察内存使用情况，
并进行相应的内存管理操作：

::

    redis> MEMORY HELP
    1) "MEMORY USAGE <key> [SAMPLES <count>] - Estimate memory usage of key"
    2) "MEMORY STATS                         - Show memory usage details"
    3) "MEMORY PURGE                         - Ask the allocator to release memory"
    4) "MEMORY MALLOC-STATS                  - Show allocator internal stats"

其中，
使用 ``MEMORY USAGE`` 子命令可以估算储存给定键所需的内存：

::

    redis> SET msg "hello world"
    OK

    redis> SADD fruits apple banana cherry
    (integer) 3

    redis> MEMORY USAGE msg
    (integer) 62

    redis> MEMORY USAGE fruits
    (integer) 375

使用 ``MEMORY STATS`` 子命令可以查看 Redis 当前的内存使用情况：

::

    redis> MEMORY STATS
    1) "peak.allocated"
    2) (integer) 1014480
    3) "total.allocated"
    4) (integer) 1014512
    5) "startup.allocated"
    6) (integer) 963040
    7) "replication.backlog"
    8) (integer) 0
    9) "clients.slaves"
    10) (integer) 0
    11) "clients.normal"
    12) (integer) 49614
    13) "aof.buffer"
    14) (integer) 0
    15) "db.0"
    16) 1) "overhead.hashtable.main"
        2) (integer) 264
        3) "overhead.hashtable.expires"
        4) (integer) 32
    17) "overhead.total"
    18) (integer) 1012950
    19) "keys.count"
    20) (integer) 5
    21) "keys.bytes-per-key"
    22) (integer) 10294
    23) "dataset.bytes"
    24) (integer) 1562
    25) "dataset.percentage"
    26) "3.0346596240997314"
    27) "peak.percentage"
    28) "100.00315093994141"
    29) "fragmentation"
    30) "2.1193723678588867"

使用 ``MEMORY PURGE`` 子命令可以要求分配器释放更多内存：

::

    redis> MEMORY PURGE
    OK

使用 ``MEMORY MALLOC-STATS`` 子命令可以展示分配器内部状态：

::

    redis> MEMORY MALLOC-STATS
    Stats not supported for the current allocator



兼容 NAT 和 Docker
---------------------

Redis 4.0 将兼容 NAT 和 Docker ，
具体的使用方法在 ``redis.conf`` 中有记载：

::

    ########################## CLUSTER DOCKER/NAT support  ########################

    # In certain deployments, Redis Cluster nodes address discovery fails, because
    # addresses are NAT-ted or because ports are forwarded (the typical case is
    # Docker and other containers).
    #
    # In order to make Redis Cluster working in such environments, a static
    # configuration where each node known its public address is needed. The
    # following two options are used for this scope, and are:
    #
    # * cluster-announce-ip
    # * cluster-announce-port
    # * cluster-announce-bus-port
    #
    # Each instruct the node about its address, client port, and cluster message
    # bus port. The information is then published in the header of the bus packets
    # so that other nodes will be able to correctly map the address of the node
    # publishing the information.
    #
    # If the above options are not used, the normal Redis Cluster auto-detection
    # will be used instead.
    #
    # Note that when remapped, the bus port may not be at the fixed offset of
    # clients port + 10000, so you can specify any port and bus-port depending
    # on how they get remapped. If the bus-port is not set, a fixed offset of
    # 10000 will be used as usually.
    #
    # Example:
    #
    # cluster-announce-ip 10.1.1.5
    # cluster-announce-port 6379
    # cluster-announce-bus-port 6380



结语
---------

关于 Redis 4.0 的主要更新就介绍到这里，
想要详细了解 Redis 4.0 各项修改的读者可以参考 Release Note ：
https://raw.githubusercontent.com/antirez/redis/4.0/00-RELEASENOTES

想要试用新版本的读者，
只要直接获取 https://github.com/antirez/redis 的 unstable 分支即可。

| 黄健宏
| 2016.12.6
