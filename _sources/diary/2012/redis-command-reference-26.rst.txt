Redis 命令参考 2.6 版本发布
===============================

经过两周紧张的工作之后， `Redis 命令参考 <http://redis.readthedocs.org>`_ 终于完成了 2.6 版本的更新。

这次更新的内容大部分来自于官方文档对 Redis 2.6 版本的更新，包括：

* Redis 2.6 版本新增的所有命令，比如 `EVAL <http://redis.readthedocs.org/en/latest/script/eval.html>`_ 、 `PTTL <http://redis.readthedocs.org/en/latest/key/pttl.html>`_ 、 `TIME <http://redis.readthedocs.org/en/latest/server/time.html>`_ 等命令的相关文档全部翻译完毕。

* 官方文档新添加的所有命令模式(pattern)，比如 INCR 命令的 `计数器模式 <http://redis.readthedocs.org/en/latest/string/incr.html#id2>`_ 和 `限速器模式 <http://redis.readthedocs.org/en/latest/string/incr.html#id3>`_ ， EXPIRE 命令的 `导航会话模式 <http://redis.readthedocs.org/en/latest/key/expire.html#id2>`_ 等等，全部翻译完毕。

* Redis 2.6 版本对旧有命令的所有改进，比如为 ``SHUTDOWN`` 命令添加 ``SAVE`` 和 ``NOSAVE`` 修饰符，为 ``INFO`` 命令设置新输出格式， ``CONFIG RESETSTAT`` 命令的变动等，以上这些命令的文档也全部更新完毕。

除此之外，命令参考项目本身也进行了不少修改，其中最重要的两个分别是：

- 命令不再按类型分页，而是每个命令各自分为一页，加快载入速度。比如在之前的版本，访问 `/string.html` 会显示所有 Redis 字符串操作命令，而现在则是通过 `/string/set.html` 来访问 `set` 命令，通过 `/string/get` 来访问 `get` 命令。

- 添加了 disqus 评论功能。

以上就是命令参考 2.6 版本的主要更新，更详细的更新细节可以参考 `项目的更新日志 <http://redis.readthedocs.org/en/latest/change_log.html#redis-2-6>`_ 。

你也可以通过访问 `redis.readthedocs.org <http://redis.readthedocs.org>`_ ，查看新版文档。

不知不觉，从 2011 年 4 月份开始进行命令参考的翻译工作，已经过去了整整一年时间，希望这个项目可以继续实践刚开始时的想法，给更多喜欢 Redis 的朋友们予以帮助。

也希望你们喜欢这个新版本，谢谢！

| huangz
| 2012.4.2
