Redis GEO 特性简介
=========================

Redis 的 GEO 特性将在 Redis 3.2 版本释出，
这个功能可以将用户给定的地理位置信息储存起来，
并对这些信息进行操作。

本文将对 Redis 的 GEO 特性进行介绍，
说明这个特性相关命令的用户，
并在最后说明如何使用这些命令去实现“查找附近的人”以及“摇一摇”这两个功能。


版本要求
---------------------------

因为 Redis 目前的稳定版本为 Redis 3.0 ，
而 GEO 特性是 Redis 3.2 版本的特性，
所以如果你想要使用这个特性的话，
那么就需要到 `Redis 的 GitHub 页面 <https://github.com/antirez/redis>`_\ 去克隆 unstable 分支
然后编译 unstable 版本的 Redis 源码，
这样才能用到 Redis GEO 特性。


添加位置和获取位置
---------------------------

为了进行地理位置相关操作，
我们首先需要将具体的地理位置记录起来，
这一点可以通过执行 ``GEOADD`` 命令来完成，
该命令的基本格式如下：

::

    GEOADD location-set longitude latitude name [longitude latitude name ...]

``GEOADD`` 命令每次可以添加一个或多个经纬度地理位置。
其中 ``location-set`` 为储存地理位置的集合，
而 ``longitude`` 、 ``latitude`` 和 ``name`` 则分别为地理位置的经度、纬度、名字。

举个例子，
以下代码展示了如何通过 ``GEOADD`` 命令，
将清远、广州、佛山、东莞、深圳等数个广东省的市添加到位置集合 ``Guangdong-cities`` 里面：

::

    redis> GEOADD Guangdong-cities 113.2099647 23.593675 Qingyuan
    1    -- 成功添加一个位置

    redis> GEOADD Guangdong-cities 113.2278442 23.1255978 Guangzhou 113.106308 23.0088312 Foshan 113.7943267 22.9761989 Dongguan 114.0538788 22.5551603 Shenzhen
    4    -- 成功添加四个位置

在将位置记录到位置集合之后，
我们可以使用 ``GEOPOS`` 命令，
输入位置的名字并取得位置的具体经纬度：

::

    GEOPOS location-set name [name ...]

比如说，
如果我们想要获取清远、广州和佛山的经纬度，
那么可以执行以下代码：

::

    redis> GEOPOS Guangdong-cities Qingyuan Guangzhou Foshan
    1) 1) "113.20996731519699"    -- 清远的经度
       2) "23.593675019671288"    -- 清远的纬度
    2) 1) "113.22784155607224"    -- 广州的经度
       2) "23.125598202060807"    -- 广州的纬度
    3) 1) "113.10631066560745"    -- 佛山的经度
       2) "23.008831202413539"    -- 佛山的纬度


计算两个位置之间的距离
---------------------------

在拥有了地理数据之后，
我们就可以基于这些数据进行各种各样的操作。
针对地理位置信息的其中一个最简单的操作，
就是计算两个位置之间的距离。

在 Redis 里面，
计算两个位置之间的距离可以通过 ``GEODIST`` 命令来实现：

::

    GEODIST location-set location-x location-y [unit]

在调用这个命令时，
用户需要给定想要计算差距的地点 ``location-x`` 和 ``location-y`` ，
以及储存这两个地点的地理位置集合。

可选参数 ``unit`` 用于指定计算距离时的单位，
它的值可以是以下单位的其中一个：

- ``m`` 表示单位为米。

- ``km`` 表示单位为千米。

- ``mi`` 表示单位为英里。

- ``ft`` 表示单位为英尺。

如果用户没有指定 ``unit`` 参数，
那么 ``GEODIST`` 默认使用米为单位。

作为例子，
以下代码展示了如何计算清远和广州之间的距离：

::

    redis> GEODIST Guangdong-cities Qingyuan Guangzhou
    "52094.433840356309"    -- 两地相距 52094 米

上面的计算结果使用了米来表示清远和广州两地的距离，
不过在表示比较长的距离时，
我们更习惯采用公里（km）作为单位。
通过显式地给定 ``km`` （千米）作为单位，
我们可以让 ``GEODIST`` 显示两个地点之间相距的公里数：

::

    redis> GEODIST Guangdong-cities Qingyuan Guangzhou km
    "52.094433840356309"    -- 两地相距 52 公里


获取指定范围内的元素
---------------------------

除了计算两地的距离之外，
另一个常见的地理位置操作就是找出特定范围之内的其他存在的地点。
比如找出地点 x 范围 100 米之内的所有地点，
找出地点 y 范围 50 公里之内的所有地点等等。

Redis 提供了 ``GEORADIUS`` 和 ``GEORADIUSBYMEMBER`` 两个命令来实现查找特定范围内地点的功能，
它们的作用一样，
只是指定中心点的方式不同：
``GEORADIUS`` 使用用户给定的经纬度作为计算范围时的中心点，
而 ``GEORADIUSBYMEMBER`` 则使用储存在位置集合里面的某个地点作为中心点。
以下是这两个命令的基本格式：

::

    GEORADIUS location-set longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [ASC|DESC] [COUNT count]

    GEORADIUSBYMEMBER location-set location radius m|km|ft|mi [WITHCOORD] [WITHDIST] [ASC|DESC] [COUNT count]

这两个命令的各个参数的意义如下：

- ``m|km|ft|mi`` 指定的是计算范围时的单位；

- 如果给定了可选的 ``WITHCOORD`` ，
  那么命令在返回匹配的位置时会将位置的经纬度一并返回；

- 如果给定了可选的 ``WITHDIST`` ，
  那么命令在返回匹配的位置时会将位置与中心点之间的距离一并返回；

- 在默认情况下，
  ``GEORADIUS`` 和 ``GEORADIUSBYMEMBER`` 的结果是未排序的，
  ``ASC`` 可以让查找结果根据距离从近到远排序，
  而 ``DESC`` 则可以让查找结果根据从远到近排序；

- ``COUNT`` 参数指定要返回的结果数量。

作为示例，
我们可以使用 ``GEORADIUSBYMEMBER`` 去找出位于广州 50 公里、 100 公里以及 150 公里以内的城市：

::

    redis> GEORADIUSBYMEMBER Guangdong-cities Guangzhou 50 km
    1) "Foshan"
    2) "Guangzhou"

    redis> GEORADIUSBYMEMBER Guangdong-cities Guangzhou 100 km
    1) "Foshan"
    2) "Guangzhou"
    3) "Dongguan"
    4) "Qingyuan"

    redis> GEORADIUSBYMEMBER Guangdong-cities Guangzhou 150 km
    1) "Foshan"
    2) "Guangzhou"
    3) "Dongguan"
    4) "Qingyuan"
    5) "Shenzhen"


示例：查找附近的人
---------------------------

好的，
在了解了 Redis GEO 特性的基本信息之后，
接下来我们该思考如何使用这些特性去解决实际的问题了。

为了让用户可以方便地找到自己附近的其他用户，
每个社交网站基本上都内置了“查找附近的人”这一功能，
通过 Redis ，
我们也可以实现同样的功能，
以下是实现该功能的伪代码：

::

    def pin(user, longitude, latitude):
        """
        记录用户的地理位置。
        """
        GEOADD('user-location-set', longitude, latitude, user)

    def find_nearby(user, n):
        """
        返回指定用户附近 n 公里的所有其他用户。
        """
        return GEORADIUSBYMEMBER('user-location-set', user, n, unit='km')


示例：摇一摇
---------------------

为了增加乐趣性，
我们可以对“查找附近的人”这一功能进行修改 —— 
程序不是返回指定范围内的所有人，
而是随机地返回指定范围内的某个人，
这也就是非常著名的“摇一摇”功能。
以下是实现该功能的伪代码：

::

    RANDOM_RADIUS = 1    # 随机查找的范围为 1 公里

    def find_random(user):
        # 获取范围内的所有其他用户
        get_all_near_users = find_nearby(user, RANDOM_RADIUS)
        # 将查找的结果从 Python 列表转换为 Python 集合
        user_set = set(get_all_near_users)
        # 然后调用 pop() 方法，从集合里面随机地移除并返回一个元素
        return user_set.pop()


效率优化
---------------------------------------

现在的 ``find_random()`` 函数可以实现“摇一摇”功能，
但它的效率并不高：
因为程序每次执行这个函数的时候都需要重新执行 ``find_nearby()`` 函数以查找用户附近的位置，
然而大部分用户的位置并不经常改变，
并且 ``GEORADIUSBYMEMBER`` 命令的执行代价并不低，
因此每次执行 ``find_random()`` 都重新执行 ``find_nearby()`` 是一种非常低效的做法。

为了优化 ``find_random()`` 的效率，
我们可以为 ``find_random()`` 的结果创建缓存：
把每个执行“摇一摇”的用户的 ``find_nearby()`` 结果储存到一个 Redis 集合里面，
并设置一个过期时间（比如 5 分钟），
然后通过对集合使用 ``SRANDMEMBER`` 来随机地获取用户。
这样用户在指定过期时间内执行的所有“摇一摇”操作都只会引起一次 ``GEORADIUSBYMEMBER`` ，
这将极大地提高 ``find_random()`` 的执行效率。

另外，
如果用户密集地聚集在一起，
那么通过使用 ``GEORADIUSBYMEMBER`` 命令提供的 ``COUNT`` 参数可以有效地减少指定范围内的用户数量，
这可以提高 ``find_nearby()`` 的效率，
从而提高 ``find_random()`` 的效率。

因为篇幅关系，
优化版的 ``find_random()`` 的具体实现这里就不给出了，
有兴趣的读者可以自己尝试完成这个函数。


Redis GEO 命令
-------------------

想要了解更多关于 ``GEOADD`` 、 ``GEORADIUS`` 等 Redis GEO 特性命令的相关信息，
请访问《Redis命令参考》： `RedisDoc.com <http://redisdoc.com>`_


结语
------------

好的，
关于 Redis GEO 特性的简单介绍就到此结束，
希望这篇文章对于大家了解 Redis 的 GEO 特性能够有所帮助，
我也期待着大家能够和我分享其他 GEO 特性的用法。

| 黄健宏（huangz）
| 2015.8.8
