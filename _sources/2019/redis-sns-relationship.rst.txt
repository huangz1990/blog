使用 Redis 储存社交关系
============================

.. note::

    本文摘录自即将出版的《Redis使用手册》，
    详情请见： `RedisGuide.com <http://RedisGuide.com>`_ 。

微博、twitter 以及类似的社交网站都允许用户通过加关注或者加好友的方式，
构建一种社交关系：
这些网站上的每个用户都可以关注其他用户，
也可以被其他用户关注。
通过正在关注名单（following list），
用户可以查看自己正在关注的用户及其人数；
而通过关注者名单（follower llist），
用户可以查看有哪些人正在关注自己，
以及有多少人正在关注自己。

代码清单 5-5 展示了一个使用集合来记录社交关系的方法：

- 程序为每个用户维持两个集合，
  一个集合储存用户的正在关注名单，
  而另一个集合则储存用户的关注者名单。

- 当一个用户（关注者）关注另一个用户（被关注者）的时候，
  程序会将被关注者添加到关注者的正在关注名单里面，
  并将关注者添加到被关注者的关注者名单里面。

- 当关注者取消对被关注者的关注时，
  程序会将被关注者从关注者的正在关注名单中移除，
  并将关注者从被关注者的关注者名单中移除。

----

代码清单 5-5 使用集合实现社交关系：\ ``/set/relationship.py``

.. literalinclude:: code/relationship.py

----

以下代码展示了社交关系程序的基本使用方法：

::

    >>> from redis import Redis
    >>> from relationship import Relationship
    >>> client = Redis(decode_responses=True)
    >>> peter = Relationship(client, 'peter')  # 这个对象记录的是 peter 的社交关系
    >>> peter.follow('jack')  # 关注一些人
    >>> peter.follow('tom')
    >>> peter.follow('mary')
    >>> peter.get_all_following()  # 获取目前正在关注的所有人
    set(['mary', 'jack', 'tom'])
    >>> peter.count_following()    # 统计目前正在关注的人数
    3
    >>> jack = Relationship(client, 'jack')    # 这个对象记录的是 jack 的社交关系
    >>> jack.get_all_follower()    # peter 前面关注了 jack ，所以他是 jack 的关注者
    set(['peter'])
    >>> jack.count_follower()      # jack 目前只有一个关注者
    1

----

图 5-10 展示了以上代码创建的各个集合。

----

图 5-10 社交关系集合示例

.. image:: image/IMAGE_RELATIONSHIP_EXAMPLE.png


