# 4. 语句（Statements）


## 1. 赋值（Assignment）

**单赋值**（single assignment）：

    a = "hello" .. "world"
    t.n = t.n + 1

**多重赋值**（multiple assignment）：

    a, b = 1, 2    -- a=1, b=2

多重赋值有以下特性：

+ 首先对 `=` 右边所有的值进行求值，然后再执行赋值操作

        x, y = y, x    -- 交换 x 和 y 的值

+ 如果变量序列的长度大于值序列的长度，则多余的变量会被赋值为`nil`

        a, b, c = 1, 2    -- a=1, b=2, c=nil

+ 如果变量序列的长度小于值序列的长度，则多余的值会被忽略

        a, b = 1, 2, 3    -- a=1, b=2, 3 被忽略

单赋值 vs 多重赋值：

+ 大多数情况下，应该使用单赋值（速度更快、可读性高）
+ 多重赋值的使用场景：交换两个变量的值、收集函数返回的多个值


## 2. 局部变量与块（Local Variables and Blocks）

使用 **local** 语句创建**局部变量**：

    local i = 1    -- i 是局部变量
    j = 2          -- j 是全局变量

**块**（block）是指控制单元（control structure）、函数（function）、代码块（chunk）的主体部分。

局部变量 vs 全局变量：

变量类型 | 作用域     | 生命周期                              | 访问速度 | 是否推荐
-------- | ---------- | ------------------------------------- | -------- | -------------------------------------------------
局部变量 | 所在块范围 | 从创建开始，到所在块结束              | 快       | 推荐（可读性高、更干净的名字空间、更早的垃圾回收）
全局变量 | 全局范围   | 从创建开始，到被赋值为`nil`或程序终止 | 慢       | 尽量少用

一个 Lua 的常见用法：

    local foo = foo

好处：

+ 与全局变量 `foo` 不同的是，局部变量 `foo` 的值不会被其他模块意外修改
+ 局部变量 `foo` 拥有更快的访问速度


## 3. 控制结构（Control Structures）

### if then else

    if a < 0 then r = 0 end

    if a < b then
      r = a
    else
      r = b
    end

    if op == "+" then
      r = a + b
    elseif op == "-" then
      r = a - b
    elseif op == "*" then
      r = a * b
    elseif op == "/" then
      r = a / b
    else
      error("invalid operation")
    end

### while

    local i = 1
    while a[i] do
      print(a[i])
      i = i + 1
    end

### repeat

    -- 打印第一个非空的输入行
    repeat
      line = io.read()
    until line ~= ""
    print(line)

### for

#### 数值型 for（numeric for）

    local a = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

    for i = 1, #a do
      print(a[i])
    end

    for i = #a, 1, -1 do
      print(a[i])
    end

对于 **数值型 for**：

    for var = exp1, exp2, exp3 do
      <something>
    end

有以下几点注意事项：

+ 三个表达式 `exp1`、`exp2`、`exp3` 只在循环开始前被求值一次
+ 控制变量 `var` 是局部变量，不要在循环体外使用它
+ 不要尝试修改控制变量的值，可以用 **break** 结束循环

#### 泛型 for（generic for）

**泛型 for** 用于遍历一个迭代器函数返回的所有值：

    -- 打印表 `t` 中的所有键/值对
    for k, v in pairs(t) do
      print(k, v)
    end

上例中，`pairs` 是一个迭代器（遍历表），类似的迭代器还有：

+ `io.lines`（遍历文件行）
+ `ipairs`（遍历序列）
+ `string.gmatch`（遍历字符串）

## 4. break、return 和 goto

### break

`break` 用于终止循环（`for`、`repeat` 或者 `while`）。

### return

`return` 用于从函数中返回结果或者结束一个函数。`return` 有一个特殊限制：它只能作为 **块**（block）的最后一个语句出现：

    -- 返回序列 a 中值 v 对应的索引
    function index(a, v)
      for i = 1, #a do
        if a[i] == v then
          print("found it")
          return i
        end
      end
      print("not found")
    end

如果想在块的中间使用`return`，你需要借助于 `do` 块：

    function foo()
      return                --<< 语法错误
      do return end         -- OK
      <其他语句>
    end

### goto

`goto` 用于跳到指定标签（`::name::`）所在的代码处。其中，标签名 name 可以是任何合法的变量名。

Lua 对 `goto` 语句做了一些限制：

+ 不能跳入一个块（block），因为块中的标签名对外不可见
+ 不能跳出一个函数（function）
+ 不能跳入一个局部变量的作用域

`goto`的典型应用是模拟实现一些 Lua 缺失的控制语句：continue、多级 break（multi-level break）、多级 continue（multi-level break）、redo、局部错误处理（local error handling）等：

    while condition do
      ::redo::
      if some_condition then
        goto redo
      end
      <一些代码>
    end

    while condition do
      if some_condition then
        goto continue
      end
      <一些代码>
      ::continue::
    end


## 练习题（Exercises）

### Q1. 大部分类似 C 语法的语言都不提供 `elseif` 构造，为什么 Lua 要提供？

因为 Lua 中没有 `switch` 语句，所以在做多路选择时，经常会用到 `elseif`。

### Q2. 列出 Lua 中无条件循环（unconditional loop）的 4 种写法。你更喜欢哪一种？

写法 1：

    while true do
      <一些代码>
    end

写法 2：

    repeat
      <一些代码>
    until false

写法 3：

    for i = 1, math.huge do
      <一些代码>
    end

写法 4：

    ::redo:: do
      <一些代码>
      goto redo
    end

个人而言，我更喜欢第 1 种写法：简洁、直接、更符合惯例。

### Q3. 很多人争论说 `repeat-until` 很少使用，因此不应该出现在像 Lua 这种精简化的语言中。你怎么看？

`repeat-until` 跟 C 语言中的 `do-while` 语句很类似，对于“要预先执行一次操作，而后再根据条件来判断是否继续”的这类事务，相比 `while` 语句而言，`repeat-until` 可以让代码更简洁一些。

个人平时写 Python 代码比较多，Python 的哲学之一是：用一种方法，最好是只有一种方法来做一件事。（There should be one-- and preferably only one --obvious way to do it.）如果遵循这样一种哲学，我认为，Lua 这种精简化的语言确实可以不提供 `repeat-until`。

### Q4. 不使用 `goto`，重写以下示例中的状态机：

    goto room1    -- 初始房间

    ::room1:: do
      local move = io.read()
      if move == "south" then goto room3
      elseif move == "east" then goto room2
      else
        print("invalid move")
        goto room1    -- 留在当前房间
      end
    end

    ::room2:: do
      local move = io.read()
      if move == "south" then goto room4
      elseif move == "west" then goto room1
      else
        print("invalid move")
        goto room2    -- 留在当前房间
      end
    end

    ::room3:: do
      local move = io.read()
      if move == "north" then goto room1
      elseif move == "east" then goto room4
      else
        print("invalid move")
        goto room3    -- 留在当前房间
      end
    end

    ::room4:: do
      print("Congratulations, you won!")
    end

重写版本：

    local room1 = {}
    local room2 = {}
    local room3 = {}
    local room4 = {}

    -- 建造房间 1
    room1.east = room2
    room1.south = room3
    room1.west = room1
    room1.north = room1

    -- 建造房间 2
    room2.east = room2
    room2.south = room4
    room2.west = room1
    room2.north = room2

    -- 建造房间 3
    room3.east = room4
    room3.south = room3
    room3.west = room3
    room3.north = room1

    -- 游戏开始
    local room = room1

    -- 游戏进行中
    repeat
      local current = room
      local move = io.read()
      room = current[move] or current
      if room == current then
        print("invalid move")
      end
    until room == room4

    -- 游戏结束
    print("Congratulations, you won!")

### Q5. 请解释为什么 Lua 要限制 `goto` 不能跳出一个函数？（提示：你应该如何正确地跳出一个函数？）

在 Lua 中，跳出一个函数的正确方式是使用 `return` 语句。

### Q6. 暂略
