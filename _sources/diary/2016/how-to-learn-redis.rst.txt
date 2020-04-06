Redis 学习路线
======================

学习和使用 Redis 一般可以分为以下四个阶段：

1. 初学者入门
2. 进阶实战
3. 理解原理
4. 贡献和开发

本文接下来将在四个小节里面分别对这四个阶段进行介绍。


初学者入门
-------------

如果你只是对 Redis 感兴趣，
又或者听别人说过一些关于 Redis 的介绍，
但是却并没有实际使用过 Redis ，
那么你就处于 Redis 初学者这一阶段。

Redis 初学者可以考虑使用\ `《Redis入门指南（第2版）》 <https://book.douban.com/subject/26419240/>`_\ 作为教程，
并辅以\ `《Redis命令参考》文档 <http://redisdoc.com/>`_\ 作为参考。

《Redis入门指南》是 ioredis 、 medis 等项目的作者 `luin <https://github.com/luin>`_ 的作品，
该书深入浅出地介绍了 Redis 的主要特性、基本命令以及使用方法，
整本书的篇幅不多，
行文简单，
很容易就能够看完。
初学者可以通过阅读这本书知道 Redis 是什么以及它能做什么。

因为篇幅所限，
《Redis入门指南》并没有对 Redis 的各个命令展开进行介绍，
因此如果读者想要进一步了解某个命令的详细用法和相关信息，
那么可以通过《Redis命令参考》进行查询。

在阅读了《Redis入门指南》和《Redis命令参考》之后，
初学者应该对 Redis 的功能、作用以及使用方法有了基本的了解，
并能够使用 Redis 去解决一些简单的问题。
在此之后，
初学者就可以向下一阶段进发，
考虑如何将 Redis 应用到实际的工作当中。

.. note:: 扩展阅读

    除了《Redis入门指南》和《Redis命令参考》之外，
    以下列出的一些资料也值得 Redis 初学者去观看和阅读：

    - 《Redis从入门到精通》课程： http://www.chinahadoop.cn/course/115

    - 《Redis课程》系列视频： http://my.tv.sohu.com/pl/9102138/index.shtml

    - Redis 官方网站上的入门介绍文章（英文，可能需要翻墙访问）： http://redis.io/topics/data-types-intro

    - 《What is Redis?》系列文章（英文，可能需要翻墙访问）： https://matt.sh/what-is-redis



进阶实战
------------

学习 Redis 的第二个阶段是进阶实战阶段，
处于这一阶段的 Redis 学习者应该对 Redis 有了基本的理解，
熟悉 Redis 各个命令以及各项特性的基本用法，
但还是不太清楚应该如何使用 Redis 去解决自己在工作上遇到的问题。

为此，
处于这一阶段的 Redis 学习者可以通过阅读\ `《Redis实战》 <http://redisinaction.com/>`_\ 一书以及其他 Redis 用户分享的心得来提高自己使用 Redis 的能力。

《Redis实战》一书是 Redis Group 讨论组中的热门发言者 `Josiah Carlson <https://github.com/josiahcarlson>`_ 所作，
该书通过实际的例子，
展示了使用 Redis 构建多种不同的应用程序的方法。
处于进阶阶段的 Redis 学习者可以通过阅读该书来学习如何使用 Redis 去构建实际的应用，
然后举一反三，
把书中介绍的程序和方法应用到自己遇到的问题上。 

除了《Redis实战》之外，
国内外的很多公司（比如twitter、新浪微博等）都在网上公布了他们使用 Redis 的方法、心得和经验，
Redis 学习者可以通过这些分享中了解到更多使用 Redis 的例子，
以及这些公司在使用 Redis 过程中遇到的问题、困难和陷阱，
从而学会如何在实际中更好地使用和管理 Redis 。

实践使用 Redis 的另一个难点是如何在大规模的数据环境中使用 Redis ，
要解决这个问题就需要对 Redis 进行扩展：
目前扩展 Redis 常见的技术包括 Redis 自带的\ `复制（replication） <http://redis.io/topics/replication>`_ 、\ `Sentinel <http://redis.io/topics/sentinel>`_  和 `Cluster <http://redis.io/topics/cluster-tutorial>`_ 功能，
以及 `twemproxy <https://github.com/twitter/twemproxy>`_ 和 `codis <https://github.com/CodisLabs/codis>`_ 等项目，
Redis 用户可以通过这些技术的相关文档来学习如何使用这些技术。



理解原理
-------------

在弄懂了如何在实际中使用 Redis 之后，
我们要考虑的就是如何解决 Redis 在使用过程中引发的问题；
如何优化 Redis 的性能；
如何对 Redis 进行二次开发，
使得它可以符合自己的某些要求；
又或者准备去开发一个自家公司特有的类 Redis 数据库。

为了达到这些目的，
我们必须对 Redis 的运作原理和内部结构有所了解。
要做到这一点，
我们必须深入地研读 Redis 的源码：https://github.com/antirez/redis 。

除了 Redis 源码之外，
一个比较好的学习 Redis 内部原理的资料就是\ `《Redis设计与实现》 <http://redisbook.com/>`_\ 一书，
并且该书也附带了一个\ `带有注释的 Redis 源码项目 <https://github.com/huangz1990/redis-3.0-annotated>`_\ 。
通过同时阅读书本和带注释的源代码，
读者能够快速地了解到 Redis 的内部构造，
以及各项主要功能的实现原理。



贡献和开发
--------------

在了解了 Redis 的原理之后，
我们可以考虑向 Redis 项目贡献代码，
又或者开发自己的类 Redis 数据库。

除了以上两点之外，
我们还可以考虑通过 Redis 最新的可载入模块系统（loadable module system），
以编写模块的方式来为 Redis 添加新功能： http://antirez.com/news/106 。


结语
------------

好的，
关于 Redis 学习资料的介绍就到此结束，
希望这些资料会对正在学习和使用 Redis 的朋友们带来帮助。

利益申明：本文作者是《Redis命令参考》和《Redis实战》的译者，《Redis设计与实现》的作者，《Redis从入门到精通》的讲师。

| 黄健宏（huangz）
| 2016.5.24
