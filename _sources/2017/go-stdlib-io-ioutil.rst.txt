Go 标准库文档翻译：io/ioutil 包
===================================


.. highlight:: go

本文是 Go 标准库中 io/ioutil 包文档的翻译，
原文地址为： 
https://golang.org/pkg/io/ioutil/


概述
--------

ioutil 包实现了一些 I/O 实用函数。


变量
---------

Discard 是一个 io.Writer ，
它的所有成功执行的 Write 操作都不会产生任何实际的效果：

::

    var Discard io.Writer = devNull(0)


NopCloser 函数
------------------

::

    func NopCloser(r io.Reader) io.ReadCloser

NopCloser 返回一个包裹起给定 Reader r 的 ReadCloser ，
这个 ReadCloser 带有一个无参数的 Close 方法。


ReadAll 函数
------------------

::

    func ReadAll(r io.Reader) ([]byte, error)

对 r 进行读取，
直到发生错误或者遇到 EOF 为止，
然后返回被读取的数据。
一次成功的读取将返回 nil 而不是 EOF 作为 err 的值：
这是因为 ReadAll 的定义就是要读取 r 直到遇到 EOF 为止，
所以它不会把读取到的 EOF 当做错误，
也不会向调用者返回它。

示例：

.. literalinclude:: code/ioutil_readall.go

示例执行结果：

.. literalinclude:: code/ioutil_readall_output.txt


ReadDir 函数
----------------

::

    func ReadDir(dirname string) ([]os.FileInfo, error)

读取 dirname 指定的目录，
并返回一个根据文件名进行排序的目录节点列表。

示例：

.. literalinclude:: code/ioutil_readdir.go

示例执行结果：

.. literalinclude:: code/ioutil_readdir_output.txt


ReadFile 函数
-------------------

::

    func ReadFile(filename string) ([]byte, error)

读取名字为 filename 的文件并返回文件中的内容。
一次成功的读取将返回 nil 而不是 EOF 作为 err 的值：
这是因为 ReadFile 的定义就是要读取整个文件，
所以它不会把读取到的 EOF 当做错误，
也不会向调用者返回它。


TempDir 函数
----------------

::

    func TempDir(dir, prefix string) (name string, err error)

在目录 dir 中新创建一个带有指定前缀 prefix 的临时目录，
然后返回该目录的路径。
如果 dir 的值是一个空字符串，
那么 TempDir 将使用 os.TempDir 中指定的目录作为创建临时文件的默认目录。

即使有多个程序同时调用 TempDir ，
TempDir 也不会创建出相同的目录。

调用者负责在使用完这个临时目录之后删除它。

示例：

.. literalinclude:: code/ioutil_tempdir.go

示例执行结果：

.. literalinclude:: code/ioutil_tempdir_output.txt


TempFile 函数
------------------

::

    func TempFile(dir, prefix string) (f *os.File, err error)

在目录 dir 新创建一个名字带有指定前缀 prefix 的临时文件，
以可读写的方式打开它，
并返回一个 \*os.File 指针。

如果 dir 的值是一个空字符串，
那么 TempFile 将使用 os.TempDir 中指定的目录作为创建临时文件的默认目录。

即使有多个程序同时调用 TempFile ，
TempFile 也不会创建出相同的文件。

调用者可以通过 f.Name() 来获取这个文件的路径名。

调用者负责在使用完这个临时文件之后删除它。

示例：

.. literalinclude:: code/ioutil_tempfile.go

示例执行结果：

.. literalinclude:: code/ioutil_tempfile_output.txt


WriteFile 函数
-------------------

::

    func WriteFile(filename string, data []byte, perm os.FileMode) error

将给定的数据 data 写入到名字为 filename 的文件里面。
如果文件不存在，
那么使用给定的权限 perm 去创建它；
如果文件已经存在，
那么 WriteFile 在对其进行写入之前会先清空文件中已有的内容。


License
----------------

Portions of this page are modifications based on work created and `shared by Google <https://developers.google.com/terms/site-policies>`_ and used according to terms described in the `Creative Commons 3.0 Attribution License <http://creativecommons.org/licenses/by/3.0/>`_ .
