《Redis实战》中的 Redis 应用示例
===============================================

.. image:: image/ria.png
   :align: right

《Redis实战》在书中介绍了很多 Redis 使用示例，
其中包括缓存、分布式锁、队列、聊天室、微博客、用户关系图谱等一系列应用。

为了方便大家学习和使用这些用例，
我在本文中将《Redis实战》各章中的用例、这些用例所在的章节以及它们的源码都一一罗列了出来，
希望大家会喜欢。

想要了解《Redis实战》一书的更多信息，
请访问书本的读者支持主页 `redisinaction.com <http://redisinaction.com>`_ 。

第 1 章
------------

- 类似 Reddit.com / HackerNews 的文章推荐网站，书本 1.3 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch01_listing_source.py#L128>`_ 。

第 2 章
-----------

- 登录 Cookies ，书本 2.1 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch02_listing_source.py#L12>`_ 。

- 购物车，书本 2.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch02_listing_source.py#L66>`_ 。

- 网页缓存，书本 2.3 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch02_listing_source.py#L101>`_ 。

- 数据行缓存，书本 2.4 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch02_listing_source.py#L124>`_ 。

第 3 章
------------

无

第 4 章
-----------

- 对日志文件进行聚合计算，书本 4.1.1 节第 2 小节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch04_listing_source.py#L29>`_ 。

- 在主从服务器中实现同步写入，书本 4.2.4 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch04_listing_source.py#L88>`_ 。

- 游戏物品拍卖行，书本 4.4.1 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch04_listing_source.py#L150>`_ 。

第 5 章
-----------

- 固定长度的先进先出日志队列，书本 5.1.1 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch05_listing_source.py#L24>`_ 。

- 根据日志的出现次数排列日志的日志队列，书本 5.1.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch05_listing_source.py#L54>`_ 。

- 使用有序集合实现计数器，书本 5.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch05_listing_source.py#L96>`_ 。

- 记录统计数据，书本 5.2.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch05_listing_source.py#L213>`_ 。

- IP 所属地查询程序，书本 5.3 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch05_listing_source.py#L316>`_ 。

- 服务器维护判断程序，书本 5.4.1 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch05_listing_source.py#L391>`_ 。

- 配置信息储存程序，书本 5.4.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch05_listing_source.py#L413>`_ 。

- 自动连接 Redis 服务器的连接器，书本 5.4.3 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch05_listing_source.py#L451>`_ 。

第 6 章
---------------

- 带有自动补全特性的联系人列表，书本 6.1 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch06_listing_source.py#L19>`_ 。

- 分布式锁，书本 6.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch06_listing_source.py#L169>`_ 。

- 带有超时限制特性的分布式锁，书本 6.2.5 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch06_listing_source.py#L251>`_ 。

- 基本计数信号量，书本 6.3.1 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch06_listing_source.py#L277>`_ 。

- 公平计数信号量，书本 6.3.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch06_listing_source.py#L309>`_ 。

- 先进先出（FIFO）队列，书本 6.4.1 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch06_listing_source.py#L382>`_ 。

- 任务优先级队列，书本 6.4.1 节第 2 小节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch06_listing_source.py#L382>`_ 。

- 延迟任务队列，书本 6.4.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch06_listing_source.py#L457>`_ 。

- 网络聊天室，书本 6.5.2 ，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch06_listing_source.py#L505>`_ 。

- 使用 Redis 进行文件分发，书本 6.6.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch06_listing_source.py#L656>`_ 。

第 7 章
--------------

- 简单的反向搜索引擎，书本 7.1 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch07_listing_source.py#L12>`_ 。

- 简单的广告定向引擎，书本 7.3 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch07_listing_source.py#L358>`_ 。

- 职位搜索引擎，书本 7.4 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch07_listing_source.py#L611>`_ 。


第 8 章
------------

- 类似 Twitter 的微博客程序，书本第 8 章，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch08_listing_source.py>`_ 。

- 储存用户信息，书本 8.1.1 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch08_listing_source.py#L105>`_ 。

- 创建 Twitter 状态消息，书本 8.1.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch08_listing_source.py#L146>`_ 。

- 创建时间线，书本 8.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch08_listing_source.py#L146>`_ 。

- 用户关系图谱（关注和被关注），书本 8.3 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch08_listing_source.py#L196>`_ 。

- 删除 Twitter 状态消息，书本 8.4 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch08_listing_source.py#L461>`_ 。

- 实现社交网站的流 API ，书本 8.5 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch08_listing_source.py#L516>`_ 。

第 9 章
------------

- 分片散列，书本 9.2.1 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch09_listing_source.py#L162>`_ 。

- 分片集合，书本 9.2.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch09_listing_source.py#L226>`_ 。

- 以二进制方式储存地理位置，书本 9.3 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch09_listing_source.py#L299>`_ 。

第 10 章
---------------

无

第 11 章
--------------

- 使用 Lua 脚本实现分布式锁，书本 11.2.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch11_listing_source.py#L166>`_ 。

- 使用 Lua 脚本实现计数信号量，书本 11.2.3 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch11_listing_source.py#L257>`_ 。

- 使用 Lua 脚本实现分片列表，书本 11.4.2 节，\ `实现源码 <https://github.com/huangz1990/riacn-code/blob/master/ch11_listing_source.py#L453>`_ 。

| 黄健宏
| 2015.11.19
