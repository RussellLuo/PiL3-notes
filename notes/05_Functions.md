5. 函数（Functions）
===================

    -- 一个名为 show 的函数
    -- 函数参数（a、b）的工作原理，与`局部变量`相同
    function show(a, b)
      print(a, b)
    end

    -- 函数对多个参数的处理规则，与`多重赋值`相同
    show(1)         -- 输出：1    nil
    show(1, 2)      -- 输出：1    2
    show(1, 2, 3)   -- 输出：1    2    (3 被忽略)

    -- 默认参数的用法
    function incr(v, n)
      n = n or 1    -- n 的默认值为 1
      return v + n
    end

    incr(10)       -- 11
    incr(10, 2)    -- 12


1. 多个结果（Multiple Results）
------------------------------

在 Lua 中，函数可以返回多个结果。

    function foo0() end                    -- 不返回结果
    function foo1() return "a" end         -- 返回 1 个结果
    function foo2() return "a", "b" end    -- 返回 2 个结果

对于返回多个结果的函数（以 `foo2` 为例）：

+ 作为语句调用时，返回的结果全部被忽略

        foo2()

+ 在表达式中调用时，只保留第一个结果

        "it is: " .. foo2()    -- "it is: a"

+ 出现在 `多重赋值`、`函数调用的参数`、`表构造` 和 `return 语句` 中，并且作为其中最后一个（或者唯一的）表达式时，返回的所有结果都会被保留

        [[ 多重赋值 ]]

        -- 唯一的表达式（OK）
        x, y = foo2()            -- x="a", y="b"

        -- 最后一个的表达式（OK）
        x, y, z = "c", foo2()    -- x="c", y="a", c="b"

        -- 不满足规定
        x, y, z = foo2(), "c"    -- x="a", y="c", z=nil

        [[ 函数调用的参数 ]]

        -- 唯一的表达式（OK）
        print(foo2())            -- 输出：a    b

        -- 最后一个的表达式（OK）
        print("c", foo2())       -- 输出：c    a    b

        -- 不满足规定
        print(foo2(), "c")       -- 输出：a    c

        [[ 表构造 ]]

        -- 唯一的表达式（OK）
        t = {foo2()}             -- t={"a", "b"}

        -- 最后一个的表达式（OK）
        t = {"c", foo2()}        -- t={"c", "a", "b"}

        -- 不满足规定
        t = {foo2(), "c"}        -- t={"a", "c"}

        [[ return 语句 ]]

        -- 唯一的表达式（OK）
        return foo2()            -- 返回："a", "b"

        -- 最后一个的表达式（OK）
        return "c", foo2()       -- 返回："c", "a", "b"

        -- 不满足规定
        return foo2(), "c"       -- 返回："a", "c"

不返回结果（`foo0`） vs 返回结果（`foo1`）：

    x = foo0()       -- x=nil
    x = foo1()       -- x="a"

    print(foo0())    -- 输出：（空）
    print(foo1())    -- 输出：a

    t = {foo0()}             -- t={} （空表）
    t = {foo1()}             -- t={"a"}
    t = {foo0(), foo1()}     -- t={nil, "a"}

    return foo0()    -- 返回：（空）
    return foo1()    -- 返回："a"

调用函数时，多加一对括号，可以强制只返回一个结果：

    (foo0())    -- nil
    (foo1())    -- "a"
    (foo2())    -- "a"

借助 `table.unpack` 可以实现：以任何序列（即不存在 `nil` 的表）作为参数，动态调用任何函数。

    local f = string.find
    local a = {"hello", "ll"}
    f(table.unpack(a))           -- 等价于：string.find("hello", "ll")


2. 可变参数函数（Variadic Functions）
------------------------------------

**可变参数函数** 是指一个函数，它可以接收个数可变的参数。例如 `print` 就是一个可变参数函数。

    -- 将参数作为结果直接返回
    function identity(...)
      return ...
    end

    identity("a", "b", "c")    -- "a", "b", "c"


参数列表中的三个点 `(...)` 表明函数 `foo` 是可变参数的（variadic）。在函数内部，可以通过表达式 `...` 来获取实际传递给函数的参数，表达式 `...` 被称为 *可变参数表达式*（vagrag expression），它的行为与“返回多个结果的函数”类似。

    -- 以下两个函数 show1 和 show2 的功能等价
    function show1(x, y, z)
      print(x, y, z)
    end

    function show2(...)
      local x, y, z = ...
      print(x, y, z)
    end

可变参数函数也可以有任意个数的固定参数，但固定参数必须出现在可变参数之前：

    function fwrite(fmt, ...)
      return io.write(string.format(fmt, ...))
    end

    fwrite()                -- fmt=nil, 没有额外参数（会报错，因为 string.format 需要一个字符串参数）
    fwrite("a")             -- fmt="a", 没有额外参数
    fwrite("%d%d", 4, 5)    -- fmt="%d%d", 额外参数：4, 5

如果可变参数中不存在 `nil`，则可以在函数内部借助 `{...}` 来收集或遍历所有参数；如果可变参数中存在 `nil`，`{...}` 就不再是一个合法的序列，此时就需要借助 `table.pack` 函数（由 `table.pack` 返回的表，其中包含一个额外的字段 `n`，它记录了函数的实际参数个数）。

    -- 可变参数中不存在 nil
    function sum(...)
      local s = 0
      for i, v in ipairs({...}) do
        s = s + v
      end
      return s
    end

    add()           -- 0
    add(1)          -- 1
    add(1, 2, 3)    -- 6

    -- 可变参数中存在 nil
    function has_nils(...)
      local arg = table.pack(...)
      for i = 1, arg.n do
        if arg[i] == nil then
          return true
        end
      end
      return false
    end

    has_nils()             -- false
    has_nils(nil)          -- true
    has_nils(1, 2)         -- false
    has_nils(1, nil, 3)    -- true

如果可变参数中不可能有 `nil` 值，则建议使用 `{...}`（而不是 `table.pack(...)`），因为它更简洁、速度更快。


3. 命名参数（Named Arguments）
-----------------------------

在 Lua 中，函数参数的传递机制都是按位置的（positional）：调用函数时，第一个实参的值传给第一个形参，以此类推。

Lua 不直接支持 **命名参数** 的语法，但是可以借助`表`来实现：将所有参数打包到一个表，然后以这个表为函数的唯一参数。

    function rename(arg)
      return os.rename(arg.old, arg.new)
    end

    rename({old="old.lua", new="new.lua"})


练习题（Exercises）
------------------

### Q1. 写一个函数，它接收任意个数的字符串，并返回这些字符串的连接。

    function concatenate(...)
      local final = ""
      for i, s in ipairs({...}) do
        final = final .. s
      end
      return final
    end

### Q2. 写一个函数，它接收一个数组，并打印出数组中的所有元素。如果使用 table.unpack，请比较优缺点。

    function print_all(array)
      print(table.unpack(array))
    end

优点：简单，缺点：？

### Q3. 写一个函数，它接收任意个数的值，并返回除第一个外的所有值。

    function all_except_first(first, ...)
      return ...
    end

### Q4. 写一个函数，它接收一个数组，并打印出数组中元素的所有组合。

暂略
