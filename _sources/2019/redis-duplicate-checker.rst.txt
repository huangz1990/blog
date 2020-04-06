使用 HyperLogLog 实现概率去重
=====================================

.. note::

    本文摘录自《Redis使用手册》，
    详情请见： `RedisGuide.com <http://RedisGuide.com>`_ 。

在构建应用程序的过程中，
我们经常需要与广告等垃圾信息做斗争。
因为垃圾信息的发送者通常会使用不同的帐号、在不同的地方发送相同的垃圾信息，
所以寻找垃圾信息的其中一种比较简单有效的手段就是找出那些重复的信息：
如果两个不同的用户发送了完全相同的信息，
又或者同一个用户重复地发送了多次完全相同的信息，
那么这些信息很有可能就是垃圾信息。

判断两段信息是否相同并不是一件容易的事情，
如果使用一般的字符串比对函数（比如 ``strcmp``\ ）来完成这一操作，
那么每当有用户尝试执行信息发布操作时，
程序就需要执行复杂度为 O(N*M) 的比对操作，
其中 N 为信息的长度，
而 M 则为系统目前已有的信息数量。
不难看出，
随着系统储存的信息越来越多，
这种比对操作将变得越来越慢，
最终成为系统的瓶颈。

为了降低鉴别重复信息所需的复杂度，
我们可以使用 HyperLogLog 来记录所有已发送的信息 ——
每当用户发送一条信息的时候，
程序就使用 ``PFADD`` 命令将这条信息添加到 HyperLogLog 里面：

- 如果命令返回 ``1`` ，
  那么这条信息就是未出现过的新信息；

- 如果命令返回 ``0`` ，
  那么这条信息就是已经出现过的重复信息。

因为 HyperLogLog 使用的是概率算法，
所以即使信息的长度非常长，
HyperLogLog 判断信息是否重复所需的时间也非常短；
另外因为 HyperLogLog 并不会随着被计数信息的增多而变大，
所以程序可以把所有需要检测的信息都记录到同一个 HyperLogLog 里面，
这使得实现重复信息检测程序所需的空间极大地减少。
代码 7-2 展示了这个使用 HyperLogLog 实现的重复信息检测程序。


----

代码 7-2 使用 HyperLogLog 实现的重复信息检测程序：\ ``/hyperloglog/duplicate_checker.py``

.. literalinclude:: code/duplicate_checker.py

----

以下代码展示了如何使用 ``DuplicateChecker`` 程序去检测并发现重复的信息：

::

    >>> from redis import Redis
    >>> from duplicate_checker import DuplicateChecker
    >>> client = Redis(decode_responses=True)
    >>> checker = DuplicateChecker(client, 'duplicate-message-checker')
    >>> checker.is_duplicated("hello world!")  # 输入一些非重复信息
    False
    >>> checker.is_duplicated("good morning!")
    False
    >>> checker.is_duplicated("bye bye")
    False
    >>> checker.unique_count()                 # 查看目前非重复信息的数量
    3
    >>> checker.is_duplicated("hello world!")  # 发现重复信息
    True

检测重复信息这个问题实际上就是算法中的“去重问题”，
因此其他去重问题也可以使用 ``DuplicateChecker`` 程序中展示的方法来解决。



