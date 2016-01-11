# 6. 函数的更多讨论（More about Functions）

在 Lua 中，函数是拥有词法作用域的第一等公民。

第一等公民是指：函数可以被赋值给变量，可以作为参数传递给其他函数，也可以作为其他函数的返回值。拥有词法作用域是指：函数可以访问其外围函数中的局部变量。

Lua 中的所有值都是匿名的（也就是没有名字），函数也不例外。通常的函数定义：

    function foo(x) return 2*x end

其实是以下定义的语法糖：

    foo = function(x) return 2*x end  -- 将匿名函数赋值给全局变量 `foo`


## 1. 闭包（Closures）

对于一个内嵌函数，其外围函数中定义的局部变量，我们称之为非局部变量（non-local variable）。例如：

    function newCounter()  -- 外围函数
      local i = 0          -- 非局部变量
      return function()    -- 内嵌函数，同时也是匿名函数（anonymous function）
        i = i + 1          -- 访问非局部变量
        return i
      end
    end

闭包是指一个函数，加上能让该函数正确访问非局部变量的结构支撑。例如：

    c1 = newCounter()
    print(c1())  --> 1
    print(c1())  --> 2

    c2 = newCounter()
    print(c2())  --> 1
    print(c1())  --> 3
    print(c2())  --> 2

根据上面的输出，我们可以得知：c1 和 c2 是基于同一个函数的两个独立的闭包。


## 2. 非全局函数（Non-Global Functoins）

因为函数是第一等公民，所以既可以把它存储到全局变量，也可以存储到表字段和局部变量。

### 存储到表字段

如果把函数存储到表字段，我们可以这样写：

    Lib = {}
    Lib.foo = function(x, y) return x + y end
    Lib.goo = function(x, y) return x - y end

    print(Lib.foo(2, 3), Lib.goo(2, 3))  --> 5    -1

也可以使用表构造器：

    Lib = {
      foo = function(x, y) return x + y end,
      goo = function(x, y) return x - y end
    }

此外，Lua 还提供了这种定义方式：

    Lib = {}
    function Lib.foo(x, y) return x + y end
    function Lib.goo(x, y) return x - y end

### 存储到局部变量

如果把函数存储到局部变量，那这个函数只能在给定作用域内可见。例如：

    local f = function(<params>)
      <body>
    end

    local g = function(<params>)
      <some code>
      f()                         -- f 在这里可见
      <some code>
    end

这个特性对 Lua 包（packages）很有用：在包内定义的局部函数，只在包内可见，这样一个包就是一个名字空间。

这种直接把匿名函数赋值给局部变量的做法，尽管在大部分情况下可行，但对于递归调用的函数，就会有问题：

    local fact = function(n)
      if n == 0 then
        return 1
      else
        return n * fact(n-1)  -- 递归调用 `fact`（这里会有问题）
      end
    end

上述代码的问题在于：当 Lua 编译到递归调用表达式 `fact(n-1)` 时，局部变量 `fact` 还没有定义，所以该表达式会尝试调用全局的 `fact`。解决办法是先定义局部变量，再定义函数：

    local fact
    fact = function(n)
      if n == 0 then
        return 1
      else
        return n * fact(n-1)  -- 执行时，`fact` 已经获取了正确的值
      end
    end

Lua 专门为这种局部函数的定义提供了一种语法糖：

    local function foo (<params>)
      <body>
    end

    -- 等价于

    local foo; foo = function(<params>) <body> end


## 3. 恰当的尾部调用（Proper Tail Calls）

Lua 可以正确地处理尾部调用（以及尾部递归）。例如：

    function foo(n)
      if n > 0 then
        return foo(n - 1)
      end
    end

不管入参 n 有多大，执行函数都绝不会导致栈溢出。


## 练习题（Exercises）

暂略
