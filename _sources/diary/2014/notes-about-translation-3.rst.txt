翻译笔记之三：消除不恰当的括号
===================================

有些作者比较喜欢使用括号来对内容进行补充说明，
但是使用不恰当或者非必要的括号常常会令内容变得复杂，
并对读者理解内容产生阻碍。

为了解决这些问题，
我在进行翻译的时候都会尽量地通过调整语句来消除括号，
争取让译文的表述变得更自然一些。

比如在《Redis in Action》第 147 页“AGGREGATING DATA LOCALLY”一节中：

    If we perform the aggregates locally on a per-country basis for the entire day 
    (being that there are around 300 countries),
    we can instead write 300 values to Redis.

这里的“(being that there are around 300 countries)”就是一个括号用得不好的例子 —— “there are around 300 countries”这个陈述应该出现在最前面，这样之后的条件“if we perform ... the entire day”和之后的结果“we can instead ... to Redis”就会来得更自然一些。

为此，
我在翻译时就将括号的内容提到了句子的前面，
这样这个句子就有完整的因果关系了。

以下为修改后的译文：

    因为只有大约 300 个国家，
    所以如果执行……，
    那么只需要对 Redis 写入大约 300 个值就可以了。

可以看到，
句子的表述在修改之后变得更自然了，
而原文要表达的意思也没有变化。

| huangz
| 2014.12.14
