使用 redis-py 储存地理位置数据
=========================================

Redis 3.2 版本的其中一个重要更新就是提供了对\ `地理位置（GEO）数据 <http://redisdoc.com/geo/index.html>`_\ 的支持，
这一特性允许用户将地理位置信息储存到 Redis 数据库中，
并对它们执行距离计算、范围查找等操作。

尽管 Redis 3.2 正式释出已经有一段时间了，
但是 Redis 最常用的 Python 库 redis-py 却一直没有添加对 GEO 特性的支持，
这给使用 Python 操作 Redis 的用户们带来了不少麻烦。

可喜的是，
今天笔者在逛 github 的时候，
发现 redis-py 的最新版本已经添加了对 GEO 特性的支持，
所以今天就让我们一起来看看如何在 redis-py 中处理地理位置数据。

.. image:: image/google-map.png


下载并安装新版 redis-py
--------------------------

因为支持 GEO 命令的最新版 redis-py 仍处于开发阶段，
所以它无法通过 pypi 取得。
为此，
我们需要从 `redis-py 的 github 页面 <https://github.com/andymccurdy/redis-py>`_\ 手动克隆最新版本的 redis-py ：

::

    $ git clone git@github.com:andymccurdy/redis-py.git

在克隆操作执行完毕之后，
我们进入到 ``redis-py`` 文件夹中，
然后将这个新版本安装到系统中：

::

    $cd redis-py
    $sudo setup.py install

如果你的系统已经安装了其他版本的 redis-py ，
那么记得在安装新版之前，
先将旧版本卸载掉。

在安装操作执行完毕之后，
我们在解释器中载入 ``redis-py`` 库：

::

    >>> from redis import Redis
    >>> r = Redis()

通过对 ``Redis()`` 对象的属性进行访问，
我们可以确认各个 GEO 命令在 redis-py 中都有了相应的方法：

::

    >>> for i in dir(r):
    ...   if i.startswith("geo"):
    ...     print(i)
    ... 
    geoadd
    geodist
    geohash
    geopos
    georadius
    georadiusbymember

接下来，
就让我们逐个试试这些方法。


添加地理位置
----------------

首先要测试的是 ``geoadd()`` 方法，
这个方法调用的是 Redis 的 `GEOADD <http://redisdoc.com/geo/geoadd.html>`_ 命令，
它的文档如下：

::

    geoadd(self, name, *values) method of redis.client.Redis instance
        Add the specified geospatial items to the specified key identified
        by the ``name`` argument. The Geospatial items are given as ordered
        members of the ``values`` argument, each item or place is formed b
        the triad latitude, longitude and name.

.. **

作为例子，
以下代码展示了如何使用 ``geoadd()`` 方法，
将清远、广州和佛山这三个城市的坐标添加到 ``"Guangdong"`` 这个键里面：

::

    >>> r.geoadd("Guangdong", "113.2099647", "23.593675", "Qingyuan", 113.2278442, 23.1255978, "Guangzhou", 113.106308, 23.0088312, "Foshan")
    3


获取地理位置
--------------

在将地理位置储存到键里面之后，
我们就可以使用 `GEOPOS <http://redisdoc.com/geo/geopos.html>`_ 命令去获取已储存的地理位置信息。
在 redis-py 里面，
``GEOPOS`` 命令可以通过执行 ``geopos()`` 方法来调用，
以下是这一方法的文档：

::

    geopos(self, name, *values) method of redis.client.Redis instance
        Return the postitions of each item of ``values`` as members of
        the specified key identified by the ``name``argument. Each position
        is represented by the pairs lat and lon.

.. **

比如说，
以下代码就展示了如何使用 ``geopos()`` 方法去从 ``"Guangdong"`` 键中获取清远和广州的地理位置：

::

    >>> r.geopos("Guangdong", "Qingyuan", "Guangzhou")
    [(113.20996731519699, 23.593675019671288), (113.22784155607224, 23.125598202060807)]


计算两地间的距离
---------------------

对于被储存的两个地理位置，
我们可以使用 `GEODIST <http://redisdoc.com/geo/geodist.html>`_ 命令去计算它们之间的距离，
而这个命令在 redis-py 中可以通过同名的 ``geodist()`` 方法去调用，
以下是该方法的文档说明：

::

    geodist(self, name, place1, place2, unit=None) method of redis.client.Redis instance
        Return the distance between ``place1`` and ``place2`` members of the
        ``name`` key.
        The units must be one o fthe following : m, km mi, ft. By default
        meters are used.

比如说，
要计算清远和广州之间的距离，
我们可以执行以下代码：

::

    >>> r.geodist("Guangdong", "Qingyuan", "Guangzhou")
    52094.4338

因为 ``GEODIST`` 命令默认使用米作为单位，
所以它返回了 ``52094.4338`` 米作为结果，
为了让这个结果更为直观一些，
我们可以将 ``GEODIST`` 命令的单位从米改为千米（公里）：

::

    >>> r.geodist("Guangdong", "Qingyuan", "Guangzhou", unit="km")
    52.0944

现在，
我们可以更直观地看到清远和广州之间相距 52.0944 公里了。


范围查找
-----------

Redis 的 `GEORADIUS 命令 <http://redisdoc.com/geo/georadius.html>`_ 和 `GEORADIUSBYMEMBER 命令 <http://redisdoc.com/geo/georadiusbymember.html>`_ 可以让用户基于指定的地理位置或者已有的地理位置进行范围查找，
redis-py 也通过同名的 ``georadius()`` 方法和 ``georadiusbymember()`` 方法来执行这两个命令。

以下是 ``georadius()`` 方法的文档：

::

    georadius(self, name, longitude, latitude, radius, unit=None, withdist=False, withcoord=False, withhash=False, count=None, sort=None, store=None, store_dist=None) method of redis.client.Redis instance
        Return the members of the of the specified key identified by the
        ``name``argument which are within the borders of the area specified
        with the ``latitude`` and ``longitude`` location and the maxium
        distnance from the center specified by the ``radius`` value.

        The units must be one o fthe following : m, km mi, ft. By default

        ``withdist`` indicates to return the distances of each place.

        ``withcoord`` indicates to return the latitude and longitude of
        each place.

        ``withhash`` indicates to return the geohash string of each place.

        ``count`` indicates to return the number of elements up to N.

        ``sort`` indicates to return the places in a sorted way, ASC for
        nearest to fairest and DESC for fairest to nearest.

        ``store`` indicates to save the places names in a sorted set named
        with a specific key, each element of the destination sorted set is
        populated with the score got from the original geo sorted set.

        ``store_dist`` indicates to save the places names in a sorted set
        named with a sepcific key, instead of ``store`` the sorted set
        destination score is set with the distance.

而以下则是 ``georadiusbymember()`` 方法的文档：

::

    georadiusbymember(self, name, member, radius, unit=None, withdist=False, withcoord=False, withhash=False, count=None, sort=None, store=None, store_dist=None) method of redis.client.Redis instance
        This command is exactly like ``georadius`` with the sole difference
        that instead of taking, as the center of the area to query, a longitude
        and latitude value, it takes the name of a member already existing
        inside the geospatial index represented by the sorted set.

作为例子，
以下代码展示了如何通过给定深圳的地理坐标（114.0538788，22.5551603）来查找位于它指定范围之内的其他城市，
这一查找操作是通过 ``georadius()`` 方法来完成的：

::

    >>> r.georadius("Guangdong", 114.0538788, 22.5551603, 100, unit="km", withdist=True)
    []  # 没有城市在深圳的半径 100 公里之内

    >>> r.georadius("Guangdong", 114.0538788, 22.5551603, 120, unit="km", withdist=True)
    [['Foshan', 109.4922], ['Guangzhou', 105.8065]]  # 佛山和广州都在深圳的半径 120 公里之内

    >>> r.georadius("Guangdong", 114.0538788, 22.5551603, 150, unit="km", withdist=True)
    [['Foshan', 109.4922], ['Guangzhou', 105.8065], ['Qingyuan', 144.2205]]  # 佛山、广州和清远都在深圳的半径 150 公里之内

另一方面，
以下代码则展示了如何通过 ``georadiusbymember()`` 方法，
找出位于广州指定半径范围内的其他城市：

::

    >>> r.georadiusbymember("Guangdong", "Guangzhou", 30, unit="km", withdist=True)
    [['Guangzhou', 0.0], ['Foshan', 17.9819]]  # 佛山在广州的半径 30 公里范围之内

    >>> r.georadiusbymember("Guangdong", "Guangzhou", 100, unit="km", withdist=True)
    [['Foshan', 17.9819], ['Guangzhou', 0.0], ['Qingyuan', 52.0944]]  # 佛山和清远都在广州的半径 100 公里范围之内


获取 geohash
----------------

最后，
用户可以通过 ``geohash()`` 方法调用 `GEOHASH <http://redisdoc.com/geo/geohash.html>`_ 命令，
以此来获得指定地理位置的 geohash 值，
以下是该方法的文档：

::

    geohash(self, name, *values) method of redis.client.Redis instance
        Return the geo hash string for each item of ``values`` members of
        the specified key identified by the ``name``argument.

.. **

作为例子，
以下代码展示了如何获取清远、广州和佛山的 geohash 值：

::

    >>> r.geohash("Guangdong", "Qingyuan", "Guangzhou", "Foshan")
    ['ws0w0phgp70', 'ws0e89curg0', 'ws06vu9s0j0']


结语
----------

好的，
这次关于使用 redis-py 处理 GEO 数据的介绍就到此结束，
希望这篇文章能够帮助大家更好地了解 redis-py 对 GEO 数据的支持，
也希望这个新版本的 redis-py 能够尽早释出，
让大家能够尽快地在 redis-py 里面用上 GEO 命令。

| 黄健宏
| 2016.8.12
