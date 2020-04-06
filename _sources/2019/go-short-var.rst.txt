Go 的简短声明
===========================

.. note::

    本文摘录自即将出版的《Go语言趣学指南》，
    请访问 `gpwgcn.com <http://gpwgcn.com>`_  以获取更多相关信息。

    .. image:: image/gpwgcn.jpg
       :scale: 80%

简短声明为 ``var`` 关键字提供了另一种备选语法。
以下两行代码是完全等效的：

.. code:: go

   var count = 10
   count := 10

初看上去，少键入两三个字符似乎不算什么，但小数怕长计，所以简短声明还是要比
``var`` 关键字流行得多。 更重要的是，简短声明还可以用在一些 ``var``
关键字无法使用的地方。

代码清单 4-2 展示了 ``for``
循环的另一种形式，它包含了初始化语句、比较条件语句以及对 ``count``
变量执行减法运算的后置语句。 在使用这种形式的 ``for``
循环时，我们需要依次向循环提供初始化语句、比较条件语句和后置语句。

--------------

代码清单 4-2 更简洁的倒数程序： ``loop.go``

.. code:: go

    var count = 0

    for count = 10; count > 0; count-- {
        fmt.Println(count)
    }

    fmt.Println(count)  // count 变量仍然处于作用域之内

--------------

在不使用简短声明的情况下，\ ``count``
变量的声明必须放置在循环之外，这意味着 ``count``
变量将在循环结束之后继续存在于作用域。

但是正如代码清单 4-3 所示，在使用简短声明的情况下，\ ``count``
变量的声明和初始化将成为 ``for``
循环的一部分，并且该变量将在循环结束之后脱离作用域，而尝试在循环之外访问
``count`` 变量将导致 Go 编译器报告 ``undefined: count`` 错误。

--------------

代码清单 4-3 在 ``for`` 循环中使用简短声明： ``short-loop.go``

.. code:: go

    for count := 10; count > 0; count-- {
        fmt.Println(count)
    }    // 随着循环结束，count 变量将不再处于作用域之内。

--------------

   {提示}

   为了代码的可读性考虑，
   声明变量的位置和使用变量的位置应该尽可能地贴近。

除了 ``for`` 循环之外，简短声明还可以在 ``if`` 语句里面声明新的变量。
比如代码清单 4-4 中的 ``num`` 变量就可以用在 ``if`` 语句的所有分支当中。

--------------

代码清单 4-4 在 ``if`` 语句中使用简短声明： ``short-if.go``

.. code:: go

    if num := rand.Intn(3); num == 0 {
        fmt.Println("Space Adventures")
    } else if num == 1 {
        fmt.Println("SpaceX")
    } else {
        fmt.Println("Virgin Galactic")
    }    // 随着 if 语句结束，num 变量将不再处于作用域之内。

--------------

正如代码清单 4-5 所示，跟 ``if`` 语言一样，简短声明也可以用在 ``switch``
语句里面。

--------------

代码清单 4-5 在 ``switch`` 语句中使用简短声明： ``short-switch.go``

.. code:: go

    switch num := rand.Intn(10); num {
    case 0:
        fmt.Println("Space Adventures")
    case 1:
        fmt.Println("SpaceX")
    case 2:
        fmt.Println("Virgin Galactic")
    default:
        fmt.Println("Random spaceline #", num)
    }

--------------

   {速查 4-5}

   如果代码清单 4-4 和 4-5 不使用简短声明， 那么 ``num``
   变量的作用域将产生何种变化？

..

   {速查 4-2 答案}

   因为 ``if`` 语句、 ``switch`` 语句和 ``for``
   语句只能使用业已声明的变量， 所以在不使用简短声明的情况下，
   程序只能在 ``if`` 等语句的前面声明 ``num`` 变量， 从而导致该变量在
   ``if`` 等语句结束之后仍然存在于作用域。


