Go 标准库文档翻译：container/list 包
===========================================

.. highlight:: go

本文是 Go 标准库中 container/list 包文档的翻译，
原文地址为： 
https://golang.org/pkg/container/list/


概述
---------

list 包实现了一个双链表（doubly linked list）。

用户可以通过以下方法来遍历链表，
其中 l 为 \*List ：

::

    for e := l.Front(); e != nil; e = e.Next() {
            // do something with e.Value
    }

示例：

.. literalinclude:: code/iterate_list.go

示例执行结果：

.. literalinclude:: code/iterate_list_output.txt


Element 类型
----------------

Element 用于代表双链表的元素：

::

    type Element struct {

        // 储存在这个元素里面的值
        Value interface{}
        
        // 其他已过滤或者未导出的字段……
    }


(\*Element) Next 方法
^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (e *Element) Next() *Element

返回链表中的下一个元素或者 nil 。


(\*Element) Prev 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (e *Element) Prev() *Element

返回链表中的前一个元素或者 nil 。


List 类型
------------

List 用于表示双链表。

空列表可以用作 List 的零值。

::

    type List struct {
        // contains filtered or unexported fields
    }

New 函数
^^^^^^^^^^^^^^

::

    func New() *List

返回一个初始化后的链表。

(\*List) Back 方法
^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) Back() *Element

返回链表的最后一个元素或者 nil 。

(\*List) Front 方法
^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) Front() *Element

返回链表的第一个元素或者 nil 。

(\*List) Init 方法
^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) Init() *List

初始化或者清空链表。

(\*List) InsertAfter 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) InsertAfter(v interface{}, mark *Element) *Element

将一个包含了值 v 的新元素 e 添加到元素 mark 的后面，
并返回 e 作为结果。

如果 mark 不是链表的元素，
那么不对链表做任何动作。

(\*List) InsertBefore 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) InsertBefore(v interface{}, mark *Element) *Element

将一个包含了值 v 的新元素 e 添加到元素 mark 的前面，
并返回 e 作为结果。

如果 mark 不是链表的元素，
那么不对链表做任何动作。

(\*List) Len 方法
^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) Len() int

返回链表包含的元素数量。
该方法的复杂度为 O(1) 。

(\*List) MoveAfter 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) MoveAfter(e, mark *Element)

将元素 e 移动至元素 mark 之后。

如果 e 或者 mark 不是链表的元素，
又或者 e == mark ，
那么不对链表做任何动作。

(\*List) MoveBefore 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) MoveBefore(e, mark *Element)

将元素 e 移动至元素 mark 之前。

如果 e 或者 mark 不是链表的元素，
又或者 e == mark ，
那么不对链表做任何动作。

(\*List) MoveToBack 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) MoveToBack(e *Element)

将元素 e 移动到链表的末尾。

如果 e 不是链表的元素，
那么不对链表做任何动作。

(\*List) MoveToFront 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) MoveToFront(e *Element)

将元素 e 移动到链表的开头。

如果 e 不是链表的元素，
那么不对链表做任何动作。

(\*List) PushBack 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) PushBack(v interface{}) *Element

将包含了值 v 的元素 e 插入到链表的末尾并返回 e 。

(\*List) PushBackList 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) PushBackList(other *List)

将链表 other 的副本插入到链表 l 的末尾。
other 和 l 可以是同一个链表。

(\*List) PushFront 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) PushFront(v interface{}) *Element

将包含了值 v 的元素 e 插入到链表的开头并返回 e 。

(\*List) PushFrontList 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) PushFrontList(other *List)

将链表 other 的副本插入到链表 l 的开头。
other 和 l 可以是同一个链表。

(\*List) Remove 方法
^^^^^^^^^^^^^^^^^^^^^^^^^

::

    func (l *List) Remove(e *Element) interface{}

如果 e 是链表 l 中的元素，
那么移除元素 e 。

这个方法会返回元素 e 的值 e.Value 作为返回值。


License
----------------

Portions of this page are modifications based on work created and `shared by Google <https://developers.google.com/terms/site-policies>`_ and used according to terms described in the `Creative Commons 3.0 Attribution License <http://creativecommons.org/licenses/by/3.0/>`_ .
