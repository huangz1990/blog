使用 Redis 实现登录会话
==================================

.. note::

    本文摘录自即将出版的《Redis使用手册》，
    详情请见： `RedisGuide.com <http://RedisGuide.com>`_ 。

为了方便用户，
网站一般都会为已登录的用户生成一个加密令牌，
然后把这个令牌分别储存在服务器端和客户端，
之后每当用户再次访问该网站的时候，
网站就可以通过验证客户端提交的令牌来确认用户的身份，
从而使得用户不必重复地执行登录操作。

另一方面，
为了防止用户因为长时间不输入密码而导致遗忘密码，
并且为了保证令牌的安全性，
网站一般都会为令牌设置一个过期期限（比如一个月），
当期限到达之后，
用户的会话就会过时，
而网站则会要求用户重新登录。

上面描述的这种使用令牌来避免重复登录的机制一般被称为登录会话（login session），
通过使用 Redis 的散列，
我们可以构建出代码清单 3-4 所示的登录会话程序。

----

代码清单 3-4 使用散列实现的登录会话程序：\ ``/hash/login_session.py``

.. literalinclude:: code/login_session.py

----

``LoginSession`` 的 ``create()`` 方法首先会计算出随机的会话令牌以及会话的过期时间戳，
然后使用用户 ID 作为字段，
将令牌和过期时间戳分别储存到两个散列里面。

在此之后，
每当客户端向服务器发送请求并提交令牌的时候，
程序就会使用 ``validate()`` 方法验证被提交令牌的正确性：
``validate()`` 方法会根据用户的 ID ，
从两个散列里面分别取出用户的会话令牌以及会话的过期时间戳，
然后通过一系列检查判断令牌是否正确以及会话是否过期。

最后，
``destroy()`` 方法可以在用户手动登出（logout）时调用，
它可以删除用户的会话令牌以及会话的过期时间戳，
让用户重新回到未登录状态。

在拥有 ``LoginSession`` 程序之后，
我们可以通过执行以下代码，
为用户 peter 创建出相应的会话令牌：

::

    >>> from redis import Redis
    >>> from login_session import LoginSession
    >>>
    >>> client = Redis(decode_responses=True)
    >>> session = LoginSession(client, "peter")
    >>>
    >>> token = session.create()
    >>> token
    '3b000071e59fcdcaa46b900bb5c484f653de67055fde622f34c255a65bd9a561'

并通过以下代码，
验证给定令牌的正确性：

::

    >>> session.validate("wrong_token")
    'SESSION_TOKEN_INCORRECT'
    >>>
    >>> session.validate(token)
    'SESSION_TOKEN_CORRECT'

然后在会话使用完毕之后，
通过执行以下代码来销毁会话：

::

    >>> session.destroy()
    >>>
    >>> session.validate(token)
    'SESSION_NOT_LOGIN'

图 3-16 展示了使用 ``LoginSession`` 程序在数据库里面创建多个会话时的样子。

----

图 3-16 登录会话程序数据结构示意图

.. image:: image/IMAGE_SESSION_HASHS.png

