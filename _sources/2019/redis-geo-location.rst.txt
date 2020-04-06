使用 Redis GEO 储存地理位置
=================================

.. note::

    本文摘录自《Redis使用手册》，
    详情请见： `RedisGuide.com <http://RedisGuide.com>`_ 。

    .. image:: image/redisguide.png

很多社交网站都提供了与地理位置相关的功能，
比如记录用户自己的位置、获取指定用户的位置，
又或者查找指定范围内的其他用户等等。

在学习了 ``GEOADD`` 、 ``GEOPOS`` 和 ``GEODIST`` 这三个命令之后，
我们同样可以构建出一个具有基本功能的用户地理位置程序，
该程序能够记录用户所在的位置、获取指定用户所在的位置以及计算两个用户之间的直线距离，
具体的实现如代码清单 9-1 所示。

----

代码清单 9-1 具有基本功能的用户地理位置程序：\ ``/geo/location.py``

.. literalinclude:: code/location.py
   :lines: 1-32

----

这个程序所做的就是使用 ``GEOADD`` 命令把用户的 ID 以及用户所在的经纬度关联起来，
然后使用 ``GEOPOS`` 命令去获取用户所在的经纬度，
又或者使用 ``GEODIST`` 命令去计算两个用户之间的直线距离。

通过执行以下代码，
我们可以创建出相应的用户位置对象，
并使用它记录 peter 、jack 和 tom 这三个用户的地理位置：

::

    >>> from redis import Redis
    >>> from location import Location
    >>> client = Redis(decode_responses=True)
    >>> location = Location(client)
    >>> location.pin("peter", 113.20996731519699, 23.593675019671288)
    >>> location.pin("jack", 113.22784155607224, 23.125598202060807)
    >>> location.pin("tom", 113.40603142976761, 22.511156445825442)

然后通过执行以下代码，
获取 peter 和 jack 的坐标：

::

    >>> location.get("peter")
    (113.20996731519699, 23.593675019671288)
    >>> location.get("jack")
    (113.22784155607224, 23.125598202060807)

最后通过执行以下代码，
计算 peter 和 jack 之间的直线距离，
以及 peter 和 tom 之间的直线距离：

::

    >>> location.calculate_distance("peter", "jack")
    52.0944
    >>> location.calculate_distance("peter", "tom")
    122.0651


