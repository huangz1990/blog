我进行翻译的方法
======================

.. image:: image/ria-cover.jpg 
   :align: left

在上一篇博文提到过，
我从今年的 2 月末开始进行《Redis in Action》（后称《RIA》）一书的翻译工作。

虽然在接下这个翻译工作之前，
我已经翻译过一些网上的文档，
比如 Redis 的文档和 Clojure 的 API 文档，
但正式地翻译整本书还是第一次，
因此，
为了把《RIA》的翻译工作做好，
最近我花了很多时间去思考和总结关于翻译工作的技巧和方法。

这篇文章将向大家介绍我进行翻译时的三个步骤，
并给出一些我认为对翻译工作有所帮助的小技巧，
希望这些内容能给其他进行翻译工作的朋友带来一些帮助，
也籍此来给《RIA》中文版的未来读者一个了解本书诞生过程的机会。

另外，
因为文中提到的方法和技巧都是实验性质的，
如果各位有更好的方法或者建议的话，
欢迎随时提出，
我也欢迎各位随时与我讨论关于翻译和写作方面的问题。


阅读原文
-------------

我进行翻译的第一个步骤是阅读原文，
并在阅读过程中进行以下工作：

1. 如果在原文里面发现自己不懂的单词或者语法，
   那么通过查字典或者阅读语法书来弄懂它们：
   我使用\ `爱词霸 <http://www.iciba.com/>`_\ 和 `Google 翻译 <http://translate.google.cn/>`_\ 来查单词，
   使用\ `《朗文英语语法》 <http://book.douban.com/subject/1003087/>`_\ 来查阅语法知识。
   如果原文里面出现了一些自己不认识的技术名词，
   那么 `Google <https://www.google.com/>`_ 和\ `维基百科 <http://zh.wikipedia.org/wiki/Wikipedia:%E9%A6%96%E9%A1%B5>`_\ 当然也是必不可少的。

2. 在一些能够了解基本意思，
   但暂时还没办法很好地用中文表达出来的句子底下划线，
   提醒自己在将来翻译的时候注意这些句子：
   被划线的句子一般都是比较复杂的长句，
   或者在句子里面包含了容易弄错的转折关系，
   等等。
   根据情况，
   我还会在划线句子旁边添加注释，
   以便为将来的翻译工作提供一些帮助。

3. 如果原文句子非常短的话，
   我会直接将译文写在句子的旁边，
   避免之后进行重复翻译，
   浪费时间。

**这几个步骤的主要作用在于找出自己不懂或者不太确定的地方，
为之后的翻译工作扫清障碍，
并留下一些线索和提示，
以便让之后的翻译工作更好地进行。**

下图展示了我在阅读《RIA》其中一章时，
在纸书上留下的笔记，
如果要翻译的是电子文档的话，
我通常会将阅读笔记写在空白的 A4 纸上面。

.. image:: image/page-note.png

在进行翻译之前阅读原文的另一个好处是，
译者可以在翻译时将原文的大致内容印在自己的脑海里面，
这在翻译那些带有引用内容的原文时，
可以给出更好的处理方法。

举个例子，
假设原文有一句话说“本书会在之后的小节里面再次回顾这个问题”，
如果译者事先已经阅读过原文的话，
那么译者就会知道原文所说的那个“小节”到底在书本的哪个地方，
并且在有需要的时候，
更灵活地处理译文：
比如说，
译者可以将原文翻译成“本书会在之后的 3.4 节再次回顾这个问题”，
这样译文的意思就更清晰，
也更确定，
而这种优化对于没有完整地看过原文的译者来说是很难做到的。


进行翻译
--------------

在阅读完原文之后，
我就会开始着手进行翻译，
并进行录入原文和断句操作。

录入原文指的是将要翻译的原文输入到编辑器里面（可以从电子书里面直接复制，或者通过手打来输入纸书的内容），
而断句指的则是将原文的每一段分为多个句子，
而每个句子又分为多个子句。

举个例子，
在翻译《RIA》第三章时，
我会将第三章的全部文字复制到编辑器里面，
然后在编辑器里面对原文的文字进行断句操作。

比如说，
以下是《RIA》第三章的其中一段：

    So far we’ve gone through the five structures that Redis provides, as well as shown a bit
    of pub/sub. The commands in this section are commands that operate on multiple
    types of data. We’ll first cover SORT, which can involve STRINGs, SETs or LISTs, and
    HASHes all at the same time. We’ll then cover basic transactions with MULTI and EXEC,
    which can allow you to execute multiple commands together as though they were just
    one command. Finally, we’ll cover the variety of automatic expiration commands for
    automatically deleting unnecessary data.

我会对这一段中的每个句子进行分行：

    So far we’ve gone through the five structures that Redis provides, as well as shown a bit of pub/sub. 

    The commands in this section are commands that operate on multiple types of data. 

    We’ll first cover SORT, which can involve STRINGs,  SETs or LISTs, and HASHes all at the same time. 

    We’ll then cover basic transactions with MULTI and EXEC, which can allow you to execute multiple commands together 
    as though they were just one command. 

    Finally, we’ll cover the variety of automatic expiration commands for automatically deleting unnecessary data.

然后再将每个句子分为多个子句：

    | So far
    | we’ve gone through the five structures that Redis provides,
    | as well as shown a bit of pub/sub.

    | The commands in this section
    | are commands that operate on multiple types of data.

    | We’ll first cover SORT,
    | which can involve STRINGs, SETs or LISTs, and HASHes 
    | all at the same time.

    | We’ll then cover basic transactions with MULTI and EXEC,
    | which can allow you to execute multiple commands together
    | as though they were just one command.

    | Finally,
    | we’ll cover the variety of automatic expiration commands
    | for automatically deleting unnecessary data.

如果我们将最开始的一段原文和断句之后的原文进行对比，
就可以明显地发现断句带来的好处：

- 原文是一整段长长的、密密麻麻的文字，
  别说翻译了，
  眼睛光是扫描这段文字就已经够吃力的了，
  一不小心的话就可能会看错或者看漏单词和句子。

- 相反地，
  断句之后的原文在物理上被分成了多个句子，
  眼睛扫描起来毫不费力，
  看得清楚了，
  翻译起来也事半功倍。

- 另外，
  因为我们将每个句子分成了多个子句，
  这样在翻译的时候就可以以子句为单位进行翻译，
  而不是以整个句子为单位，
  或者以整个段落为单位，
  这种化繁为简、分而治之的方法在翻译一些长句的时候非常有帮助。

根据我自己的经验，
**断句可能是翻译过程中最考技术的部分了 ——
它在很大程度上直接决定了译文的准确性：
如果译者在断句上出现了错误，
那么说明译者没有正确理解句子的意思，
而这样翻译出来的译文肯定不会是正确的。**


阅读译文
--------------

在对原文进行翻译并得出译文之后，
我会从头到尾完整地阅读一遍译文，
原因在于：

- 在进行翻译的时候，
  译者关注的通常是一个段落，
  或者一个句子，
  虽然在翻译的时候也会阅读译文并进行修改，
  但是译者却很少会完整地阅读整篇译文。

- 而在翻译完成之后阅读译文，
  可以让译者将角色变为读者，
  并以读者的角度去审视整篇译文的翻译质量和流畅程度。

虽然阅读译文只是一个非常简单的步骤，
但是对于译文的流畅度却有非常大的影响。
俗话说得好 ——
当局者迷，
旁观者清：
**很多在进行翻译时（以译者角度来看译文时）不会发现的问题，
在以读者身份阅读译文的时候却会立即显现出来。**

以我自己的经验来说，
在进行翻译的时候，
我有很多次都认为一些地方自己已经处理得足够好了，
但是当我作为一个读者去阅读译文的时候，
我又发现了一些可以进一步改善译文的地方，
而如果我自己不读译文，
又或者只在翻译的时候阅读一小段译文的话，
那么我是不会发现译文的这些缺陷的。

除了发现译文的缺陷之外，
阅读译文时要做的另一件事是修改那些带有外文味道的译文，
让译文符合中文的行文习惯：
这个工作在大部分时候只需要修改一下译文的标点符号，
或者简单调整一下译文句子的结构，
又或者使用一些成语、谚语、常用语去替换译文里面的一些句子。
经过这些修改之后，
译文就会变得更加中文化，
阅读起来也会变得更为流畅。



结语
--------------

好的，
以上介绍的就是我最近在翻译《RIA》时所使用的方法了，
因为接下来也要继续翻译《RIA》的剩余部分，
所以如果有关于翻译方面的新想法的话，
我会继续写博文来和大家分享的。

| huangz
| 2014.4.17
