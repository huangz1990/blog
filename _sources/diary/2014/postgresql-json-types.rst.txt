PostgreSQL 数据库中的 JSON 数据类型
=====================================

PostgreSQL 数据库在本月的 18 号发布了新的 9.4 版本，
该版本的亮点之一就是增强了对 JSON 数据类型的支持，
并添加了新的 JSONB 类型。

在\ `新版本的发布说明中 <http://www.postgresql.org/about/news/1557/>`_\ ，
PostgreSQL 官方这样描述 JSONB 数据类型：

    PostgreSQL 新添加的 JSONB 数据类型使得用户可以同时拥有关系型数据库和非关系型数据库，
    而不必再在这两种数据库之间选择其中一种。

    JSONB 支持快速查找（fast lookup）和基于泛化反向索引（Generalized Inverted Indexes，GIN）的简单表达式搜索查询（simple expression search query）。

    除此之外，
    用户还可以使用新添加的多个辅助函数（support function）来提取和操作 JSON 数据，
    这些函数的性能和目前最流行的文档数据库不相上下，
    甚至有过之而无不及。

    JSONB 的出现使得表格数据和文档数据可以在整个数据库环境中融为一体，和谐共处。
      
那么，JSONB 类型到底是何方神圣？
它的作用是否又如 PostgreSQL 官方所说的那样强大呢？
为了解答这个问题，也为了对 JSONB 进行全面的了解，
huangz 决定对 `PostgreSQL 9.4 版本手册（英文） <http://www.postgresql.org/docs/9.4/interactive/datatype-json.html>`_ 中的 `8.14. JSON Types <http://www.postgresql.org/docs/9.4/interactive/datatype-json.html>`_ 一节和 `9.15. JSON Functions and Operators <http://www.postgresql.org/docs/9.4/interactive/functions-json.html>`_ 一节进行翻译，
并与大家共享。

接下来，
就让我们随着文档， 
一起来探究 PostgreSQL 的 JSON 数据类型以及 JSONB 数据类型吧：

- 阅读 8.14 节《JSON Types》译文： http://pgcn.huangz.me/part2/chp8/section14.html
- 阅读 9.15 节《JSON Functions and Operators》译文： http://pgcn.huangz.me/part2/chp9/section15.html

| huangz
| 2014.12.27
