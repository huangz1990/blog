Redis 5 新功能介绍：ZPOPMAX、ZPOPMIN 以及它们的阻塞变种
===============================================================

有序集合（SortedSet又称zset）是 Redis 的老牌数据结构之一，
它的 API 一直以来都是很稳定的，
基本上没有发生过变化，
但随着 Redis 5 的到来，
新版本 Redis 也给它加上了四个新的命令，
这些命令分别是：

- `ZPOPMAX <https://redis.io/commands/bzpopmax>`_ 

- `ZPOPMIN <https://redis.io/commands/zpopmin>`_ 

- `BZPOPMAX <https://redis.io/commands/bzpopmax>`_

- `BZPOPMIN <https://redis.io/commands/bzpopmin>`_

其中，
``ZPOPMAX`` 命令用于移除并弹出有序集合中分值最大的 ``count`` 个元素：

::

    ZPOPMAX key [count]
    summary: Remove and return members with the highest scores in a sorted set
    since: 5.0.0
    group: sorted_set

而 ``ZPOPMIN`` 命令则用于移除并弹出有序集合中分值最小的 ``count`` 个元素：

::

    ZPOPMIN key [count]
    summary: Remove and return members with the lowest scores in a sorted set
    since: 5.0.0
    group: sorted_set

如果用户没有显式地为 ``count`` 参数指定值，
那么命令默认将移除并弹出一个元素。

以下是这两个命令的使用示例：

::

    127.0.0.1:6379> ZADD zset 1.0 "one" 2.0 "two" 3.0 "three"
    (integer) 3

    127.0.0.1:6379> ZPOPMAX zset
    1) "three"
    2) "3"

    127.0.0.1:6379> ZPOPMIN zset
    1) "one"
    2) "1"

``BZPOPMAX`` 和 ``BZPOPMIN`` 是上述两个命令的阻塞变种，
这两个命令每次只能弹出单个元素，
但可以接受多个键作为被弹出的对象，
并且需要使用 ``timeout`` 参数去指定命令的最长阻塞时间：

::

    BZPOPMAX key [key ...] timeout
    summary: Remove and return the member with the highest score from one or more sorted sets, or block until one is available
    since: 5.0.0
    group: sorted_set

    BZPOPMIN key [key ...] timeout
    summary: Remove and return the member with the lowest score from one or more sorted sets, or block until one is available
    since: 5.0.0
    group: sorted_set

``BZPOPMAX`` 命令和 ``BZPOPMIN`` 命令的行为跟 ``BLPOP`` 、 ``BRPOP`` 等命令的语义非常相似，
熟悉上述两个命令的读者应该不会对这两个新命令感到陌生。

| 黄健宏
| 2018.6.15
