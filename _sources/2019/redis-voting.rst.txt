使用 Redis 实现投票功能
===========================

.. note::

    本文摘录自即将出版的《Redis使用手册》，
    详情请见： `RedisGuide.com <http://RedisGuide.com>`_ 。

问答网站、文章推荐网站、论坛这类注重内容质量的网站上通常都会提供投票功能，
用户可以通过投票来支持一项内容或者反对一项内容：

- 一项内容获得的支持票数越多，
  它就会被网站安排到越显眼的位置，
  使得网站的用户可以更快速地浏览到高质量的内容。

- 与此相反，
  一项内容获得的反对票数越多，
  它就会被网站安排到越不显眼的位置，
  甚至被当作广告或者无用内容而被隐藏起来，
  使得用户可以忽略这些低质量的内容。

根据网站性质的不同，
不同的网站可能会为投票功能设置不同的称呼，
比如有些网站可能会把“支持”和“反对”叫做“推荐”和“不推荐”，
而有些网站可能会使用“喜欢”和“不喜欢”来表示“支持”和“反对”，
诸如此类，
但这些网站的投票功能在本质上都是一样的。

作为示例，
图 5-8 展示了 StackOverflow 问答网站的一个截图，
这个网站允许用户对问题及其答案进行投票，
从而帮助用户发现高质量的问题和答案。

----

图 5-8 StackOverflow 网站的投票示例，图中所示的问题获得了 10 个推荐

.. image:: image/stackoverflow.png

----

代码清单 5-4 展示了一个使用集合实现的投票程序：
对于每一项需要投票的内容，
这个程序都会使用两个集合来分别储存投支持票的用户以及投反对票的用户，
然后通过对这两个集合执行命令来实现投票、取消投票、统计投票数量、获取已投票用户名单等功能。

----

代码清单 5-4 使用集合实现的投票程序，用户可以选择支持或者反对一项内容：\ ``/set/vote.py``

.. literalinclude:: code/vote.py

----

以下代码展示了如何使用这个投票程序去记录一个问题的投票信息：

::

    >>> from redis import Redis
    >>> from vote import Vote
    >>> client = Redis(decode_responses=True)
    >>> question_vote = Vote(client, 'question::10086')  # 记录问题的投票信息
    >>> question_vote.vote_up('peter')    # 投支持票
    True
    >>> question_vote.vote_up('jack')
    True
    >>> question_vote.vote_up('tom')
    True
    >>> question_vote.vote_down('mary')  # 投反对票
    True
    >>> question_vote.vote_up_count()    # 统计支持票数量
    3
    >>> question_vote.vote_down_count()  # 统计反对票数量
    1
    >>> question_vote.get_all_vote_up_users()    # 获取所有投支持票的用户
    {'jack', 'peter', 'tom'}
    >>> question_vote.get_all_vote_down_users()  # 获取所有投反对票的用户
    {'mary'}

图 5-9 展示了这段代码创建出的两个集合，
以及这两个集合包含的元素。

----

图 5-9 投票程序创建出的两个集合

.. image:: image/IMAGE_VOTE_EXAMPLE.png

----


