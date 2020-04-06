使用 Redis 构建短网址生成程序
==================================

.. note::

    本文摘录自即将出版的《Redis使用手册》， 详情请见： `RedisGuide.com <http://RedisGuide.com>`_ 。

为了给用户提供更多发言空间，
并记录用户在网站上的链接点击行为，
大部分社交网站都会将用户输入的网址转换为相应的短网址。
比如说，
如果我们在新浪微博发言时输入网址 http://redisdoc.com/geo/index.html ，
那么微博将把这个网址转换为相应的短网址 http://t.cn/RqRRZ8n ，
当用户访问这个短网址时，
微博在后台就会对这次点击进行一些数据统计，
然后再引导用户的浏览器跳转到 http://redisdoc.com/geo/index.html 上面。

创建短网址本质上就是要创建出短网址 ID 与目标网址之间的映射，
并在用户访问短网址时，
根据短网址的 ID 从映射记录中找出与之相对应的目标网址。
比如在前面的例子中，
微博的短网址程序就将短网址 http://t.cn/RqRRZ8n 中的 ID 值 RqRRZ8n 映射到了 http://redisdoc.com/geo/index.html 这个网址上面：
当用户访问短网址 http://t.cn/RqRRZ8n 时，
程序就会根据这个短网址的 ID 值 RqRRZ8n ，
找出与之对应的目标网址 http://redisdoc.com/geo/index.html ，
并将用户引导至目标网址上面去。

作为示例，
图 3-8 展示了几个微博短网址 ID 与目标网址之间的映射关系。

----

图 3-8 微博短网址映射关系示例

.. image:: image/IMAGE_URL_MAPPING.png

----

因为 Redis 的散列正好就非常适合用来储存短网址 ID 与目标网址之间的映射，
所以我们可以基于 Redis 的散列实现一个短网址程序，
代码清单 3-1 展示了一个这样的例子。

----

代码清单 3-1 使用散列实现的短网址程序：\ ``/hash/shorty_url.py``

.. literalinclude:: code/shorty_url.py

----

代码清单 3-2 将 10 进制数字转换成 36 进制数字的程序：\ ``/hash/base36.py``

.. literalinclude:: code/base36.py

----

``ShortyUrl`` 类的 ``shorten()`` 方法负责为输入的网址生成短网址 ID ，
它的工作包括以下四个步骤：

1. 为每个给定的网址创建一个 10 进制数字 ID 。

2. 将 10 进制数字 ID 转换为 36 进制，
   并将这个 36 进制数字用作给定网址的短网址 ID ，
   这种方法在数字 ID 长度较大时可以有效地缩短数字 ID 的长度。
   代码清单 3-2 展示了将数字从 10 进制转换成 36 进制的 ``base10_to_base36`` 函数的具体实现。

3. 将短网址 ID 和目标网址之间的映射关系储存到散列里面。

4. 向调用者返回刚刚生成的短网址 ID 。

另一方面，
``restore()`` 方法要做的事情和 ``shorten()`` 方法正好相反：
它会从储存着映射关系的散列里面取出与给定短网址 ID 相对应的目标网址，
然后将其返回给调用者。

以下代码简单地展示了使用 ``ShortyUrl`` 程序创建短网址 ID 的方法，
以及根据短网址 ID 获取目标网址的方法：

::

    >>> from redis import Redis
    >>> from shorty_url import ShortyUrl
    >>> client = Redis(decode_responses=True)
    >>> shorty_url = ShortyUrl(client)
    >>> shorty_url.shorten("RedisGuide.com")  # 创建短网址 ID
    '1'
    >>> shorty_url.shorten("RedisBook.com")
    '2'
    >>> shorty_url.shorten("RedisDoc.com")
    '3'
    >>> shorty_url.restore("1")  # 根据短网址 ID 查找目标网址
    'RedisGuide.com'
    >>> shorty_url.restore("2")
    'RedisBook.com'

图 3-9 展示了上面这段代码在数据库中创建的散列结构。

----

图 3-9 短网址程序在数据库中创建的散列结构

.. image:: image/IMAGE_SHORTY_URL_EXAMPLE.png
