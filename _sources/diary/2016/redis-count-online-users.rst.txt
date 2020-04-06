使用 Redis 统计在线用户人数
==================================

在构建应用的时候，
我们经常需要对用户的一举一动进行记录，
而其中一个比较重要的操作，
就是对在线的用户进行记录。

本文将介绍四种使用 Redis 对在线用户进行记录的方案，
这些方案虽然都可以对在线用户的数量进行统计，
但每个方案都有一些自己特有的操作，
并且各个方案的性能特征以及资源消耗也各有不同。

.. image:: image/online_users.png


方案 1 ：使用有序集合
-----------------------

每当一个用户上线时，
我们就执行 `ZADD <http://redisdoc.com/sorted_set/zadd.html>`_ 命令，
将这个用户以及它的在线时间添加到指定的有序集合中：

::

    ZADD "online_users" <user_id> <current_timestamp>

通过使用 `ZSCORE <http://redisdoc.com/sorted_set/zscore.html>`_ 命令检查指定的用户 ID 在有序集合中是否有相关联的分值，
我们可以知道该用户是否在线：

::

    ZSCORE "online_users" <user_id> 

而通过执行 `ZCARD <http://redisdoc.com/sorted_set/zcard.html>`_ 命令，
我们可以知道总共有多用户在线：

::

    ZCARD "online_users"

使用有序集合储存在线用户的强大之处在于，
它是本文介绍的所有方案当中，
能够执行最多聚合操作的一个方案，
原因在于，
这一方案既可以通过有序集合的成员（也即是用户的 ID）进行聚合操作，
也可以根据有序集合的分值（也即是用户的登录时间）进行聚合操作。

首先，
通过 `ZINTERSTORE <http://redisdoc.com/sorted_set/zinterstore.html>`_ 和 `ZUNIONSTORE <http://redisdoc.com/sorted_set/zunionstore.html>`_ 命令，
我们可以对多个记录了在线用户的有序集合进行聚合计算：

::

    # 计算出 7 天之内都有上线的用户，并将它储存到 7_days_both_online_users 有序集合当中
    ZINTERSTORE 7_days_both_online_users 7 "day_1_online_users" "day_2_online_users" ... "day_7_online_users"

    # 计算出 7 天之内总共有多少人上线了
    ZUNIONSTORE 7_days_total_online_users 7 "day_1_online_users" ... "day_7_online_users"

此外，
通过 `ZCOUNT <http://redisdoc.com/sorted_set/zcount.html>`_ 命令，
我们可以统计出在指定的时间段之内有多少用户在线，
而 `ZRANGEBYSCORE <http://redisdoc.com/sorted_set/zrangebyscore.html>`_ 命令则可以让我们获取到这些用户的名单：

::

    # 统计指定时间段内上线的用户数量
    ZCOUNT "online_users" <start_timestamp> <end_timestamp>

    # 获取指定时间段内上线的用户名单
    ZRANGEBYSCORE "online_users" <start_timestamp> <end_timestamp> WITHSCORES
    
通过这一方法，
我们可以知道网站在不同时间段的上线人数以及上线用户名单，
比如说，
我们可以用这个方法来分别获知网站在早晨、上午、中午、下午和夜晚的上线人数。


方案 2 ：使用集合
------------------------

正如上一节所说，
使用有序集合能够同时储存在线用户的名单以及各个用户的上线时间，
但如果我们只想要记录在线用户的名单，
而不想要储存用户的上线时间，
那么也可以使用集合来代替有序集合，
对在线的用户进行记录。

在这种情况下，
每当一个用户上线时，
我们就执行以下 `SADD <http://redisdoc.com/set/sadd.html>`_ 命令，
将它添加到在线用户名单当中：

::

    SADD "online_users" <user_id>

通过使用 `SISMEMBER <http://redisdoc.com/set/sismember.html>`_ 命令，
我们可以检查一个指定的用户当前是否在线：

::

    SISMEMBER "online_users" <user_id>

而统计在线人数的工作则可以通过执行 `SCARD <http://redisdoc.com/set/scard.html>`_ 命令来完成：

::

    SCARD "online_users"

通过集合运算操作，
我们可以像有序集合方案一样，
对不同时间段或者日期的在线用户名单进行聚合计算。
比如说，
通过 `SINTER <http://redisdoc.com/set/sinter.html>`_ 或者 `SINTERSTORE <http://redisdoc.com/set/sinterstore.html>`_ 命令，
我们可以计算出一周都有在线的用户：

::

    SINTER "day_1_online_users" "day_2_online_users" ... "day_7_online_users"

此外，
通过 `SUNION <http://redisdoc.com/set/sunion.html>`_ 命令或者 `SUNIONSTORE <http://redisdoc.com/set/sunionstore.html>`_ 命令，
我们可以计算出一周内在线用户的总数量：

::

    SUNION "day_1_online_users" "day_2_online_users" ... "day_7_online_users"

而通过执行 `SDIFF <http://redisdoc.com/set/sdiff.html>`_ 命令或者 `SDIFFSTORE <http://redisdoc.com/set/sdiffstore.html>`_ 命令，
我们可以知道哪些用户今天上线了，
但是昨天没有上线：

::

    SDIFF "today_online_users" "yesterday_online_users"

又或者工作日上线了，
但是假日没有上线：

::

    # 计算工作日上线名单
    SINTERSTORE "weekday_online_users" "monday_online_users" "tuesday_online_users" ... "friday_online_users"
    # 计算假日上线名单
    SINTERSTORE "holiday_online_users" "saturday_online_users" "sunday_online_users"
    # 计算工作日上线但是假日未上线的名单
    SDIFF "weekday_online_users" "holiday_online_users"

诸如此类。


方案 3 ：使用 HyperLogLog
-------------------------------

虽然使用有序集合和集合能够很好地完成记录在线人数的工作，
但以上这两个方案都有一个明显的缺点，
那就是，
这两个方案耗费的内存会随着被统计用户数量的增多而增多：
如果你的网站用户数量比较多，
又或者你需要记录多天/多个时段的在线用户名单并进行聚合计算，
那么这两个方案可能会消耗你大量内存。

另一方面，
在有些情况下，
我们只想要知道在线用户的人数，
而不需要知道具体的在线用户名单，
这时有序集合和集合储存的信息就会显得多余了。

在需要尽可能地节约内存并且只需要知道在线用户数量的情况下，
我们可以使用 HyperLogLog 来对在线用户进行统计：
HyperLogLog 是一个概率算法，
它可以对元素的基数进行估算，
并且每个 HyperLogLog 只需要耗费 12 KB 内存，
对于用户数量非常多但是内存却非常紧张的系统，
这一方案无疑是最佳之选。

在这一方案下，
我们使用 `PFADD <http://redisdoc.com/hyperloglog/pfadd.html>`_ 命令去记录在线的用户：

::

    PFADD "online_users" <user_id>

使用 `PFCOUNT <http://redisdoc.com/hyperloglog/pfcount.html>`_ 命令获取在线人数：

::

    PFCOUNT "online_users"

因为 HyperLogLog 也提供了计算交集的 `PFMERGE <http://redisdoc.com/hyperloglog/pfmerge.html>`_ 命令，
所以我们也可以用这个命令计算出多个给定时间段或日期之内，
上线的总人数：

::

    # 统计 7 天之内总共有多少人上线了
    PFMERGE "7_days_both_online_users" "day_1_online_users" "day_2_online_users" ... "day_7_online_users"
    PFCOUNT "7_days_both_online_users"


方案 4 ：使用位图（bitmap）
-------------------------------

回顾上面介绍的三个方案，
我们可以得出以上结论：

- 使用有序集合或者集合能够储存具体的在线用户名单，
  但是却需要消耗大量的内存；

- 而使用 HyperLogLog 虽然能够有效地减少统计在线用户所需的内存，
  但是它却没办法准确地记录具体的在线用户名单。

那么是否存在一种既能够获得在线用户名单，
又可以尽量减少内存消耗的方法存在呢？
这种方法的确存在 ——
使用 Redis 的位图就可以办到。

Redis 的位图就是一个由二进制位组成的数组，
通过将数组中的每个二进制位与用户 ID 进行一一对应，
我们可以使用位图去记录每个用户是否在线。

当一个用户上线时，
我们就使用 `SETBIT <http://redisdoc.com/string/setbit.html>`_ 命令，
将这个用户对应的二进制位设置为 1 ：

::

    # 此处的 user_id 必须为数字，因为它会被用作索引
    SETBIT "online_users" <user_id> 1

通过使用 `GETBIT <http://redisdoc.com/string/getbit.html>`_ 命令去检查一个二进制位的值是否为 1 ，
我们可以知道指定的用户是否在线：

::

    GETBIT "online_users" <user_id>

而通过 `BITCOUNT <http://redisdoc.com/string/bitcount.html>`_ 命令，
我们可以统计出位图中有多少个二进制位被设置成了 1 ，
也即是有多少个用户在线：

::

    BITCOUNT "online_users"

跟集合一样，
用户也能够对多个位图进行聚合计算 ——
通过 `BITOP <http://redisdoc.com/string/bitop.html>`_ 命令，
用户可以对一个或多个位图执行逻辑并、逻辑或、逻辑异或或者逻辑非操作：

::

    # 计算出 7 天都在线的用户
    BITOP "AND" "7_days_both_online_users" "day_1_online_users" "day_2_online_users" ... "day_7_online_users"

    # 计算出 7 在的在线用户总人数
    BITOP "OR" "7_days_total_online_users" "day_1_online_users" "day_2_online_users" ... "day_7_online_users"

    # 计算出两天当中只有其中一天在线的用户
    BITOP "XOR" "only_one_day_online" "day_1_online_users" "day_2_online_users"

HyperLogLog 方案记录一个用户是否在线需要花费 1 个二进制位，
对于用户数为 100 万的网站来说，
使用这一方案只需要耗费 125 KB 内存，
而对于用户数为 1000 万的网站来说，
使用这一方案也只需要花费 1.25 MB 内存。

虽然位图节约内存的效果不及 HyperLogLog 那么显著，
但是使用位图可以准确地判断一个用户是否上线，
并且能够像集合和有序集合一样，
对在线用户名单进行聚合计算。
因此对于想要尽量节约内存，
但又需要准确地知道用户是否在线，
又或者需要对用户的在线名单进行聚合计算的应用来说，
使用位图可以说是最佳之选。


总结
--------

以下表格总结了以上四个方案的特点：

+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
| 方案          | 特点                                                                                                                                      |
+===============+===========================================================================================================================================+
| 有序集合      | 能够同时储存在线用户的名单以及用户的上线时间，能够执行非常多的聚合计算操作，但是耗费的内存也非常多。                                      |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
| 集合          | 能够储存在线用户的名单，也能够执行聚合计算，消耗的内存比有序集合少，但是跟有序集合一样，这个方案消耗的内存也会随着用户数量的增多而增多。  |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
| HyperLogLog   | 无论需要统计的用户有多少，只需要耗费 12 KB 内存，但由于概率算法的特性，只能给出在线人数的估算值，并且也无法获取准确的在线用户名单。       |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+
| 位图          | 在尽可能节约内存的情况下，记录在线用户的名单，并且能够对这些名单执行聚合操作。                                                            |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------+

因为 Redis 同时支持多种数据结构，
所以一个问题常常可以在 Redis 里面找多种不同的解法，
并且每种解法都有各自的优点和缺点，
本文介绍的问题就是一个很好的例子。

关于统计在线用户的方法就介绍到这里，
希望这些方案会给大家带来帮助和启发。

想要知道更多有趣和实用的 Redis 用法，
请关注我的新书《Redis使用教程》（\ `RedisGuide.com <http://RedisGuide.com>`_\ ）。

| 黄健宏（huangz）
| 2016.8.20
