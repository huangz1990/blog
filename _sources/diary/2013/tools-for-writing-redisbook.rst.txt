编写《Redis 设计与实现》时用到的工具
===========================================

在《\ `Redis 设计与实现 <http://www.redisbook.com>`_\ 》（后面简称 《设计与实现》）发布以后，
除了和 Redis 相关的问题之外，
读者们问得最多的就是制作这本书使用了什么工具。

这篇文章将对编写《设计与实现》时用到的所有工具进行介绍，
并附上一些我个人的使用心得和经验。


文档生成：Sphinx
------------------------

《设计与实现》的网页本身是由 `Sphinx <http://sphinx-doc.org/ext/graphviz.html>`_ 生成的。

.. image:: image/sphinx-log.png

Sphinx 是一个静态文档生成器，
它的源码文件由 `reStructuredText <http://docutils.sourceforge.net/rst.html>`_ （简称 rst）格式写成，
rst 格式和 markdown 格式一样，
都是一种易懂、易学的标记语言（markup language）。

Sphinx 这个项目本身是作为 `Python 官方文档 <http://docs.python.org/3/>`_ 的生成器而出现的，
它具备了一系列强大的特性，
使得这个工具非常利于编写文档：

- 通过 `Pygments <http://pygments.org/>`_ ，支持多种语言、多种风格的代码高亮。

- 代码和文件可以内嵌在 rst 文件中，也可以 `从其他文件载入 <http://sphinx-doc.org/markup/code.html?highlight=include#includes>`_ ，这保证了多个文件之间的模块化：比如你可以将一系列文档放在一个文件夹，而将文档中用到的代码放在另一个文件夹，这样不同文件之间的就不会互相干扰。

- 文档本身可以非常简单地 `引用同一文档的其他章节、其他小节，或者文档的任何地方 <http://sphinx-doc.org/markup/inline.html#role-ref>`_ 。

  更为强大的是， `一个 Sphinx 文档还可以引用另一个 Sphinx 文档的内容 <http://sphinx-doc.org/ext/intersphinx.html>`_ ：举个例子，在《设计与实现》中，我需要添加大量链向《\ `Redis 命令参考 <http://redis.readthedocs.org>`_\ 》的链接，比如 `EVAL <http://redis.readthedocs.org/en/latest/script/eval.html>`_ 。

  如果每次都要为这些命令手动添加链接，那可就真是既无聊又痛苦 —— 而使用 Sphinx 的跨文档链接就可以轻松地解决这个问题：我只要按照 Sphinx 的格式在 rst 文件中写下一个（简短的）命令，Sphinx 就会自动帮我添加到链向《Redis 命令参考》的链接，非常简单。

- Sphinx 还支持 `一系列插件 <http://sphinx-doc.org/extensions.html>`_ ，通过这些插件，可以增强 Sphinx 本身的功能。

  这些插件很多都是和 Python 有关的，但有一些在非 Python 的文档项目中都能用得上。

  比如《设计与实现》就使用了其中的 `mathjax 插件 <http://sphinx-doc.org/ext/math.html#module-sphinx.ext.mathjax>`_ 来显示 LaTeX 格式的算法复杂度；文章接下来会介绍的 Graphviz 图片生成工具，也有 `相应的 Sphinx 插件 <http://sphinx-doc.org/ext/graphviz.html>`_ ；另外，前面提到的，对跨文档链接的支持，也是通过插件来完成的。

- Sphinx `内置了多个蛮不错的外观样式 <http://sphinx-doc.org/theming.html>`_ ，更换不同的样式、或者自己 `修改样式 <http://sphinx-doc.org/templating.html>`_ 都非常简单。《设计与实现》所使用的样式就是通过修改内置的 pyramid 样式而来的，可以在 GitHub 找到这个样式： `github.com/huangz1990/der <https://github.com/huangz1990/der>`_ 。

在编写《设计与实现》之前，
我就已经用 Sphinx 做了多个文档，
比如《\ `Redis 命令参考 <http://redis.readthedocs.org/>`_\ 》和《\ `SICP 解题集 <http://sicp.readthedocs.org>`_\ 》，
等等。
所以在开始编写《设计与实现》的时候，
我也毫不犹豫地选择了 Sphinx 来作为书本的生成工具，
和以前一样，
这次 Sphinx 也很好地帮助我完成了书本的编写工作。

强烈推荐喜欢写作或者编写文档的人去尝试接触一下 Sphinx ，
说不定你也会因此喜欢上 Sphinx ，
甚至会和我一样，
变成一个文档编写爱好者。


源码的管理和托管：Git 、 GitHub 以及 Bitbucket
-------------------------------------------------------

有了书本原稿之后，
如何妥善地保管这些原稿就成了一个非常重要的问题的 ——
幸运的是，
这个问题已经基本上被 `git <http://git-scm.com/>`_ 和它的一系列托管商解决了。

.. image:: image/git-logo.png

《设计与实现》的原稿使用 Git 来进行记录和追踪，
每次编写完一部分内容之后提交，
然后推送到托管商上，
这样就不必担心原稿因为电脑出问题或者误操作而丢失了（在电脑和 git 普及以前的世界，这样的事情并不少见）。

至于源码托管商方面，
《设计与实现》的源码目前托管在 GitHub ，
不过在发布之前，
源码是作为私有项目放在 Bitbucket 上的。

.. image:: image/github-logo.png
.. image:: image/bitbucket-logo.png

`Github <https://github.com/>`_ 和 `Bitbucket <https://bitbucket.org>`_ 两个网站都拥有良好的商业支持，
所以将书稿放托管在这两个网站上，
安全性方面是可以保障的。
这两个网站在项目托管方面的主要区别是：

- GitHub 的私有项目需要按月收费，而 Bitbucket 则为至多 5 人的团队提供免费的私有项目。

- 但是 GitHub 的人气比 Bitbucket 要高，项目放在 GitHub 会受到更多关注。

如果想节约一些私有项目的托管费用，
但又想项目获得更多关注，
可以像《设计与实现》一样，
在编写书本时，
先将项目托管在 Bitbucket ，
然后等正式发布的时候，
再挪到 GitHub 上去。

另外，
因为 Bitbucket 的私有项目提供 5 人的成员名额，
所以不妨邀请一些朋友作为文档的技术复检员，
让他们加入到项目中，
请他们为你未发布的文档提供意见。


图片生成：Graphviz
-----------------------------------------

《设计与实现》的图片都是由 `Graphviz <http://graphviz.org/>`_ 生成的。

.. image:: image/graphviz-logo.png

Graphviz 是一个开源的图片可视化工具，
它和其他图片工具的最大区别是，
Graphviz 的图片是用文本来表示的 ——
也即是说，
使用者先用 dot 语言将图片以文本的方式“写”出来，
然后再用 Graphviz 将 dot 文件转换成图片：

.. image:: image/graphviz-show.png

作为例子，
上面的这个示意图就是用以下源码写出来的：

.. literalinclude:: image/graphviz-show.dot

Graphviz 生成的图片不是“画”出来的，
而是像编程一样写出来的，
这种做法有两个明显的好处：

1) 因为“画图”是由 Graphviz 负责的，所以你只需要负责定义“这个图片应该是什么样子”，至于“怎么画这个图”，就交给 Graphviz 来思考就可以了。

2) 因为图片的源码是文本，所以源码中的内容可以重用，并且这些源码文件可以用 git 来管理。

多得这两个好处，
在编写《设计与实现》时，
图片的绘制和管理方面的任务都可以轻松地完成。

不过，
从另一方面来说，
Graphviz 提供了大量的功能和参数，
要真正使用好也是要费一些功夫的。
目前《设计与实现》的图片还是非常简单的，
也不够美观，
很多复杂的图片也只能用 ASCII 文字来表示，
希望在将来的新版本中，
能用 Graphviz 创建出更多更好看的图片。


其他工具
-----------------------------------------

《设计与实现》是通过阅读并提取 Redis 源码中的设计思想来创作的，
我使用 VIM 阅读和注释源码。
（至于我是如何阅读 Redis 源码的，可以参考 `InfoQ 上对《Redis 设计与实现》的报道 <http://www.infoq.com/cn/news/2013/03/redis-book>`_ ，我在里面有说到。）

有时候为了了解代码的行为，
我会修改并对代码进行测试。
测试代码使用的是源码本身配套的、用 TCL 语言编写的 test suit ：
每次做了实验性的修改之后，
我都会 ``make test`` 一下，
看会产生什么结果。

测试代码没有用到 GDB 之类的调试工具：
``make test`` 、 Redis 的断言函数，
以及服务器打印的 log 输出，
已经能很好地满足我的需求。

我用圆珠笔和空白的 A4 纸捕捉有趣的想法，
并记录任务 TODO 。

另外，
之前曾打算购买一块大白板和一些记事贴来处理一些大小不适合使用 A4 纸来记录的信息，
不过后来就把这事给忘记了，
写这篇文章的时候忽然回忆到了，
得赶紧去买才行。


Sphinx 项目托管： ReadTheDocs
-----------------------------------------

当书本完成之后，
我们需要将 Sphinx 生成的文档发布出去，
才能让书本让更多人看到。

Sphinx 可以生成 HTML 格式的网页，
看样子我们可以找一个支持静态网页的网站来存放这些网页，
不过，
更好也更酷的办法是直接使用 ReadTheDocs 。

`ReadTheDocs <https://readthedocs.org/>`_ 是一个由多家网站赞助支持，
提供免费 Sphinx 项目托管的网站。

.. image:: image/readthedocs-logo.png

你只要在 ReadTheDocs 注册一个帐号，
然后填写你的 Sphinx 项目的相关信息（比如项目在 GitHub 上的地址），
ReadTheDocs 就会自动为你的文档生成一个特有的子域名，
比如 `redisbook.readthedocs.org <https://redisbook.readthedocs.org>`_ ，
然后任何人只要访问这个子域名，
就可以观看你的文档了。

每次当你更新你的文档源码时，
ReadTheDocs 都会自动帮你重新编译文档，
让你的更新立即展现在读者面前，
整个过程完全是自动化的。

更棒的是，
ReadTheDocs 还提供了子域名绑定功能，
你可以将你的文档和你名下的域名进行绑定，
从而为你的文档提供一个更简洁的访问地址。
比如 `www.redisbook.com <http://www.redisbook.com/>`_ 。

另外，
ReadTheDocs 也直接支持 `Google Analytics 访问统计 <https://www.google.com/analytics>`_ ，
你只要填入你的 analytics 跟踪代码，
就可以对你的文档的访问情况进行记录了。

要将 Sphinx 文档发布到网上，
ReadTheDocs 应该是你的不二之选。


结尾
---------------------

好的，以上介绍的就是编写《Redis 实现与设计》时用到的所有工具了。

关于工具的选择和使用上几乎每个人都有不同的想法，
如果不是这样的话，
反而会让我觉得奇怪。

我使用这些工具并不是说它们是最好的，
只是我习惯使用它们，
或者偶然接触到这些工具了。

如果你有关于编写书本/文档方面的好想法，
或者有更好的工具可以介绍，
欢迎在评论中留下你的意见。

最后，
如果这篇文章介绍的工具能帮上你的忙，
那就太好了。

| huangz
| 2013.4.15
