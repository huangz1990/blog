使用 Redis 构建先进先出队列解决秒杀问题
======================================================

.. note::

    本文摘录自即将出版的《Redis使用手册》，
    详情请见： `RedisGuide.com <http://RedisGuide.com>`_ 。

先进先出队列（first in first out queue）是一种非常常见的数据结构，
一般都会包含入队（enqueue）和出队（dequeue）这两个操作，
其中入队操作会将一个元素放入到队列中，
而出队操作则会从队列中移除最先被入队的元素。

先进先出队列的应用非常广泛，
它见诸于各式各样的应用程序当中。
举个例子，
很多电商网站都会在节日时推出一些秒杀活动，
这些活动会放出数量有限的商品供用户抢购，
秒杀系统的一个特点就是在短时间内会有大量用户同时进行相同的购买操作，
如果使用事务或者锁去实现秒杀程序，
那么就会因为锁和事务的重试特性而导致性能低下，
并且由于重试的存在，
成功购买商品的用户可能并不是最早执行购买操作的用户，
因此这种秒杀系统实际上是不公平的。

解决上述问题的其中一个方法就是把用户的购买操作都放入到先进先出队列里面，
然后以队列方式处理用户的购买操作，
这样程序就可以在不使用锁或者事务的情况下实现秒杀系统，
并且得益于先进先出队列的特性，
这种秒杀系统可以按照用户执行购买操作的顺序来判断哪些用户可以成功执行购买操作，
因此它是公平的。

代码清单 4-1 展示了一个使用 Redis 列表实现先进先出队列的方法。

----

代码清单 4-1 使用列表实现的先进先出队列：\ ``/list/fifo_queue.py``

.. literalinclude:: code/fifo_queue.py

----

作为例子，
我们可以通过执行以下代码，
载入并创建一个先进先出队列：

::

    >>> from redis import Redis
    >>> from fifo_queue import FIFOqueue
    >>> client = Redis(decode_responses=True)
    >>> q = FIFOqueue(client, "buy-request")

然后通过执行以下代码，
将三个用户的购买请求依次放入到队列里面：

::

    >>> q.enqueue("peter-buy-milk")
    1
    >>> q.enqueue("john-buy-rice")
    2
    >>> q.enqueue("david-buy-keyboard")
    3

最后，
按照先进先出顺序，
依次从队列中弹出相应的购买请求：

::

    >>> q.dequeue()
    'peter-buy-milk'
    >>> q.dequeue()
    'john-buy-rice'
    >>> q.dequeue()
    'david-buy-keyboard'

可以看到，
队列弹出元素的顺序跟元素入队时的顺序是完全相同的：
最先是 ``"peter-buy-milk"`` 元素，
接着是 ``"john-buy-rice"`` 元素，
最后是 ``"david-buy-keyboard"`` 元素。


