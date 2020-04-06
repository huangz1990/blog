Go 标准库文档翻译：hash 包
================================

.. highlight:: go

本文是 Go 标准库中 hash 包文档的翻译，
原文地址为： 
https://golang.org/pkg/hash/


概述
---------

hash 包提供了实现散列函数所需的接口。


Hash 类型
--------------

Hash 是实现散列函数所需的公共接口。

::

    type Hash interface {
        // 通过 Write 方法将更多数据添加到正在运行的散列当中。
        // 这个方法不会返回错误。
        // Write 方法通过嵌入 io.Writer 接口来实现。
        io.Writer

        // 将当前散列追加至 b 的末尾，并返回结果切片。
        // 这一操作不会改变底层散列的状态。
        Sum(b []byte) []byte

        // 将散列重置至初始化状态。
        Reset()

        // 返回 Sum 会返回的字节数。
        Size() int

        // 返回散列的底层块大小。
        // Write 必须能够接受任何大小的数据，
        // 但如果写入的数据量总是块大小的某个倍数的话，
        // 那么写入就会变得更为高效。
        BlockSize() int
    }


Hash32 类型
---------------

Hash32 是实现 32 位散列函数所需的公共接口。

::

    type Hash32 interface {
        Hash
        Sum32() uint32
    }


Hash64 类型
--------------

Hash64 是实现 64 位散列函数所需的公共接口。

::

    type Hash64 interface {
        Hash
        Sum64() uint64
    }


License
----------------

Portions of this page are modifications based on work created and `shared by Google <https://developers.google.com/terms/site-policies>`_ and used according to terms described in the `Creative Commons 3.0 Attribution License <http://creativecommons.org/licenses/by/3.0/>`_ .
