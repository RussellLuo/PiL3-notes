7. 迭代器与泛型 for（Iterators_and_the_Generic_for）
===================================================


1. 迭代器与闭包（Iterators and Closures）
----------------------------------------

**迭代器** 是这样一个构造（construction）：它允许你迭代（iterate）一个集合的所有元素。在 Lua 中，通常用 *函数* 来表示迭代器：每次调用函数，都会返回集合中的 “下一个” 元素。

迭代器需要在连续的调用之间记住状态，**闭包** 为此提供了绝佳的机制。

    -- 一个简单的值迭代器
    function values(t)
      local i = 0    -- 非局部变量（non-local variables）
      return function()
        i = i + 1
        return t[i]
      end
    end

    -- 普通的 while 循环
    local t = {10, 20, 30}
    local iter = values(t)      -- 创建迭代器
    while true do
      local element = iter()    -- 调用迭代器
      if element == nil then    -- 如果迭代器返回 nil，则停止循环
        break
      end
      print(element)
    end

    -- 为迭代而生的泛型 for 循环
    -- 功能等价（为你包办了上面 while 循环中需要处理的各种杂事）
    local t = {10, 20, 30}
    for element in values(t) do
      print(element)
    end


2. 泛型 for 的语义（The Semantics of the Generic for）
-----------------------------------------------------

上述迭代器的一个缺点是：每次初始化一个新的循环时，都需要创建一个新的闭包。大部分情况下，这不成问题；但在一些性能要求高的场合，这会影响性能，此时就要用到泛型 for 用于保存状态的机制。

泛型 for 的语法如下：

    for <var-list> in <exp-list> do
      <body>
    end

其中，`var-list` 是逗号分隔的变量列表，`exp-list` 是逗号分隔的表达式列表。变量列表中的第一个变量称为 *控制变量*（control variable），一旦它的值为 `nil`，就会停止循环。

实际上，泛型 for 保存了三个值：迭代器函数、不变状态（invariant state）、控制变量。执行 for 语句会经历以下步骤：

1. 求值表达式列表，以得到三个值：迭代器函数、不变状态、控制变量的初值（赋值规则类似 *多重赋值*）
2. 以不变状态、控制变量为参数，调用迭代器函数
3. 得到迭代器函数的返回值，并赋给变量列表中的变量
4. 如果控制变量为 `nil`，则停止循环；否则执行循环体，然后重复第 2 步


3. 无状态的迭代器（Stateless Iterators）
---------------------------------------

**无状态的迭代器** 是这样一个迭代器：它不保留任何状态，只根据 *不变状态* 和 *控制变量* 来产生下一个元素。因此，我们可以在多个循环中，重复使用同一个无状态的迭代器，从而避免创建新的闭包带来的成本。

`ipairs` 就是一个典型的无状态的迭代器，它的 Lua 版本实现如下：

    -- 定义迭代器
    -- 其中：表 t 作为不变状态，索引 i 作为控制变量
    local function iter(t, i)
      i = i + 1
      local v = t[i]
      if v then
        return i, v
      end
    end

    function ipairs(t)
      return iter, t, 0
    end

    -- 使用迭代器
    t = {"one", "two", "three"}
    for i, v in ipairs(t) do
      print(i, v)
    end

`pairs` 也是一个无状态的迭代器，只不过它使用的迭代器函数是内置的 `next` 函数：

    -- 定义迭代器
    -- 其中：表 t 作为不变状态，表 t 的键作为控制变量（初值为 nil）
    function pairs(t)
      return next, t, nil
    end

    -- 使用迭代器
    t = {"one", "two", "three"}
    for k, v in pairs(t) do
      print(k, v)
    end


4. 具有复杂状态的迭代器（Iterators with Complex States）
-------------------------------------------------------

多数情况下，迭代器需要保存更多状态，而不只是不变状态和控制变量。有两种实现方式：

+ 使用闭包
+ 将所需状态打包成表，然后以 `表` 作为不变状态


5. 真正的迭代器（True Iterators）
--------------------------------

Lua 中的迭代器其实更像是 “生成器”（generator），它只提供用于迭代的值，而没有执行迭代操作。

Lua 中还可以创建另一种迭代器：它会真正执行迭代操作，并在每次迭代中调用一个函数，而这个函数就是迭代器函数的参数。这种迭代器在老版本的 Lua 中很常用。

结论是：推荐使用 “生成器” 风格的迭代器。


练习题（Exercises）
------------------

### Q1. 写一个无状态的迭代器 fromto，使得下面两个循环等价：

    for i in fromto(n ,m) do
      <body>
    end

    for i = n, m do
      <body>
    end

无状态的迭代器 fromto：

    local function iter(max, current)
      current = current + 1
      if current <= max then
        return current
      end
    end

    function fromto(min, max)
      return iter, max, min - 1
    end

### Q2. 为上一题的迭代器添加一个 步幅（step）参数，并仍然实现为无状态的迭代器。

带有步幅参数的 fromto：

    local function iter(state, current)
      current = current + state.step
      if current <= state.max then
        return current
      end
    end

    function fromto(min, max, step)
      local step = step or 1
      local state = {max=max, step=step}
      return iter, state, min - step
    end

### Q3. 写一个迭代器 uniquewords，它从指定文件中返回所有不重复的单词。

    function uniquewords()
      local line = io.read()
      local pos = 1
      local words = {}

      return function()
        while line do
          local s, e = string.find(line, "%w+", pos)
          if s then
            pos = e + 1
            local word = string.sub(line, s, e)
            if not words[word] then
              words[word] = true
              return word
            end
          else
            line = io.read()
            pos = 1
          end  -- if s then
        end  -- while line do
        return nil
      end
    end

### Q4. 写一个迭代器，它从指定字符串中返回所有非空的子字符串。

    function substrings(str)
      local length = #str
      local from = 1
      local to = 0

      local function iter()
        to = to + 1
        if to > length then
          from = from + 1
          to = from
        end
        local substr = string.sub(str, from, to)
        if #substr ~= 0 then    -- 非空子串
          return substr
        end
      end

      return iter
    end

    -- 使用迭代器 substrings
    local str = "sample"
    for s in substrings(str) do
      print(s)
    end

其实，该功能的最简单的实现，是使用数值型 for 循环：

    local str = "sample"
    local length = #str
    for from = 1, length do
      for to = from, length do
        local substr = string.sub(str, from, to)
        print(substr)
      end
    end
