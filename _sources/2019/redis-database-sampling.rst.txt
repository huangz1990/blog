Redis 数据库取样程序
=========================

.. note::

    本文摘录自即将出版的《Redis使用手册》，
    详情请见： `RedisGuide.com <http://RedisGuide.com>`_ 。

在使用 Redis 的过程中，
我们可能会想要知道 Redis 数据库中各种键的类型分布状况：
比如说，
我们可能会想要知道数据库里面有多少个字符串键、有多少个列表键、有多少个散列键，
以及这些键在数据库键的总数量中占多少个百分比。

代码清单 11-2 展示了一个能够计算出以上信息的数据库取样程序。
``DbSampler`` 程序会对数据库进行迭代，
使用 ``TYPE`` 命令获取被迭代键的类型并对不同类型的键实施计数，
最终在迭代完整个数据库之后，
打印出相应的取样结果。 

----

代码清单 11-2 数据库取样程序：\ ``/database/db_sampler.py``

.. literalinclude:: code/db_sampler.py

----

以下代码展示了这个数据库取样程序的使用方法：

::

    >>> from redis import Redis
    >>> from create_random_type_keys import create_random_type_keys
    >>> from db_sampler import DbSampler
    >>> client = Redis(decode_responses=True)
    >>> create_random_type_keys(client, 1000)  # 创建 1000 个类型随机的键
    >>> sampler = DbSampler(client)
    >>> sampler.sample()
    Sampled 1000 keys.
    String: 179 keys, 17.9% of the total.
    List: 155 keys, 15.5% of the total.
    Hash: 172 keys, 17.2% of the total.
    Set: 165 keys, 16.5% of the total.
    SortedSet: 161 keys, 16.1% of the total.
    Stream: 168 keys, 16.8% of the total.

可以看到，
取样程序遍历了数据库中的一千个键，
然后打印出了不同类型键的具体数量以及它们在整个数据库中所占的百分比。

为了演示方便，
上面的代码使用了 ``create_random_type_keys()`` 函数来创建出指定数量的类型随机键，
代码清单 11-3 展示了这个函数的具体定义。

----

代码清单 11-3 随机键生成程序：\ ``/database/create_random_type_keys.py``

.. literalinclude:: code/create_random_type_keys.py

----



