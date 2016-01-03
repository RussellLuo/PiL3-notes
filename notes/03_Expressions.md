# 3. 表达式（Expressions）

Lua 中的表达式涉及到数值常量、字符串常量、变量、一元和二元运算符、函数调用，以及（非常规的）函数定义和表构造。


## 1. 数值运算符（Arithmetic Operators）

Lua 支持常见的数值运算符，其中包括：

运算符 | 类型 | 含义 | 示例
------ | ---- | ---- | ------------------------------
`+`    | 二元 | 加法 | a + b
`-`    | 二元 | 减法 | a - b
`*`    | 二元 | 乘法 | a * b
`/`    | 二元 | 除法 | a / b
`^`    | 二元 | 求幂 | x ^ 0.5, x ^ (-1/3)
`%`    | 二元 | 求余 | a % b, 5 % 2
`-`    | 一元 | 负号 | -a, -2

其中，求余运算符的定义如下：

    a % b == a - math.floor(a/b)*b

如果操作数是整数，求余操作没什么不同；如果操作数是实数，求余操作还有其他作用，比如：

- 对于实数 `x`，`x%1` 是 `x` 的小数部分，`x - x%1` 则是 `x` 的整数部分
- 类似地，`x - x%0.01` 是保留两位小数的 `x`，于是：

        x = math.pi
        print(x - x%0.01)  --> 3.14


## 2. 关系运算符（Relational Operators）

Lua 提供了下列关系运算符：

运算符 | 类型 | 含义       | 适用的运算对象         | 示例
------ | ---- | ---------- | ---------------------- | --------------------------------
`<`    | 二元 | 小于       | 两个数字，或两个字符串 | 1 < 2 (true), "a" < "b" (true)
`>`    | 二元 | 大于       | 两个数字，或两个字符串 | 2 > 1 (true), "b" > "a" (true)
`<=`   | 二元 | 小于或等于 | 两个数字，或两个字符串 | 1 <= 2 (true), "a" <= "b" (true)
`>=`   | 二元 | 大于或等于 | 两个数字，或两个字符串 | 2 >= 1 (true), "b" >= "a" (true)
`==`   | 二元 | 等于       | 任意两个值             | 1 == 2 (false), 1 == 1 (true)
`~=`   | 二元 | 不等于     | 任意两个值             | 1 ~= 2 (true), 1 ~= 1 (false)

关系运算符的运算结果总是 `boolean` 类型的值。需要特别说明的是：

1. 相对性判断（`==`、`~=`）支持所有值的比较

    - 如果两个值的类型不同，则直接被视为不相等。例如：

            print(1 == "1")  --> false

    - 对于表（table）和用户数据（userdata），只有两个值指向同一个对象时，才会被视为相等。例如：

            a = {}
            b = {}
            c = a

            print(a == b)  --> false
            print(a == c)  --> true

    - nil 只等于它自己（nil 常量，或值为 nil 的变量）。例如：

            a = nil

            print(a == nil)  --> true
            print(nil == nil)  --> true

2. 顺序判断（`<`、`>`、`<=`、`>=`）只支持两个数字，或者两个字符串的比较

    - 字符串的比较按照字母顺序进行
    - 如果比较一个数字和一个字符串（如 `1 > "1"`），Lua 会直接报错


## 3. 逻辑运算符（Logical Operators）

Lua 中有三个逻辑运算符 **and**、**or** 和 **not**。逻辑运算符把布尔 **false** 和 `nil` 视为条件假，其他的值都视为条件真。

### 基本用法

对于 **and** 运算符，如果它的第一个参数为条件假，它会返回第一个参数；否则，它会返回第二个参数。例如：

    print(4 and 5)       --> 5
    print(nil and 13)    --> nil
    print(false and 13)  --> false

对于 **or** 运算符，如果它的第一个参数为条件真，它会返回第一个参数；否则，它会返回第二个参数。例如：

    print(4 or 5)      --> 4
    print(false or 5)  --> 5

**not** 运算符总是返回 `boolean` 值：

    print(not nil)      --> true
    print(not false)    --> true
    print(not 0)        --> false
    print(not not 1)    --> true
    print(not not nil)  --> false

### 短路求值

此外，**and** 和 **or** 运算符都遵循“短路求值”（short-cut evaluation），它们只在必要的时候对第二个参数进行求值。例如，表达式 `(type(v) == "table" and v.tag == "h1")` 不会报错，因为如果 `v` 不是表，Lua 就不会对 `v.tag` 进行求值。

一些有用的惯例：

- 设置默认值

    `x = x or v` 等价于 `if not x then x = v end`（`x` 被设置为 `false` 的情况除外）

- 三元条件运算符

    `a and b or c` 等价于 C 中的 `a ? b : c`。例如，两个数的最大值可以表示为 `max = (x > y) and x or y`


## 4. 字符串连接（Concatenation）

Lua 用两个点 `..` 来表示字符串连接运算符。

如果遇到数字，Lua 会把它转换字符串：

    print("Hello" .. "World")  --> Hello World
    print(0 .. 1)              --> 01
    print(000 .. 01)           --> 01

字符串在 Lua 中是不可变对象，所以连接运算符总是创建一个新字符串，而不会改变原有字符串。


## 5. 长度运算符（The Length Operator）

长度运算符 `#` 可以用在字符串和表上。用在字符串上，它会给出字符串中的字节数；用在表（准确的说，是 `序列`）上，它会给出表的长度。

在操作序列时，长度运算符提供了一些惯例用法：

    print(a[#a])   --> 打印序列的最后一个值
    a[#a] = nil    --> 删除序列的最后一个值
    a[#a + 1] = v  --> 将 v 追加到序列的末尾


## 6. 优先级（Precedence）

运算符的优先级，从高到低为：

    ^
    not  #    -（一元）
    *    /    %
    +    -
    ..
    <    >    <=    >=    ~=    ==
    and
    or

运算符的结合性：`^`、`..` 和 `-`（一元）都是右结合的，其他运算符都是左结合的。

**注意**：不确定优先级的时候，或者在复杂的表达式中，建议用明确的括号来确定优先级。


## 7. 表构造器（Table Constructors）

Lua 中，构造器是用于创建并初始化表的表达式。

    empty = {}           -- 空表
    a = {"x", "y", "z"}  -- 数组构造器：a[1] == "x"，a[2] == "y"，a[3] == "z"
    a = {x=10, y=20}     -- 记录构造器：a.x == 10，a.y == 20
    a = {                -- 混用数组和记录构造器
      name="line",       -- a.name == "line"
      {x=0, y=0},        -- a[1]
      {x=8, y=8}         -- a[2]
    }

一般来讲，用构造器创建并初始化表，比先创建空表再添加字段，来得更高效、更优雅。

但上面的数组或记录构造器，也有它的局限性：你不能指定负数索引，也不能指定非标识符的索引。为此，Lua 提供了一个更通用、更灵活（同时也更笨重的）构造方式：借助方括号，你可以明确指定任意表达式或值作为索引。例如：

    opnames = {["+"] = "add", ["-"] = "sub", ["*"] = "mul", ["/"] = "div"}

另外，在表的构造器中，你可以在最后一个字段后面加上逗号：

    a = {[1]="red", [2]="green", [3]="blue",}

特别地，你还可以使用分号来替换逗号，以划分构造器中不同的段：

    {x=10, y=45; "one", "two", "three"}


## 练习题（Exercises）

### Q1. 下列程序的输出是什么？

    for i = -10, 10 do
      print(i, i % 3)
    end

暂略（请直接在 Lua 命令行看结果）。

### Q2. 表达式 `2^3^4` 的结果是什么？那么 2^-3^4 呢？

`^` 和 `-`（一元）都是右结合的，但 `^` 的优先级高于 `-`（一元），因此:

- `2^3^4` 等价于 `2^(3^4)`，其结果是 `2.4178516392293e+24`
- `2^-3^4` 等价于 `2^(-(3^4))`，其结果是 `4.1359030627651e-25`

### Q3. 在 Lua 中，我们可以用系数列表（如 `{a0, a1, ..., an}` 来表示一个多项式 `an * x^n + ... + a1 * x^1 + a0`。写一个函数，它以多项式（以表的形式）和 x 的值为参数，并返回该多项式的值。

这个多项式函数的一种实现：

    function calc(p, x)
      local result = 0
      for i = 1, #p do
        result = result + p[i] * (x ^ (i - 1))
      end
      return result
    end

### Q4. 最多使用 `n` 次加法和 `n` 次乘法，并且不使用求幂运算，你能重新实现上述函数吗？

不使用求幂运算的版本：

    function calc(p, x)
      local result = 0
      local e = 1
      for i = 1, #p do
        result = result + p[i] * e
        e = e * x
      end
      return result
    end

### Q5. 不使用 `type` 函数，如何检测一个值是否为 `boolean` 类型？

使用关系运算符和逻辑运算符，可以很轻易地实现：

    function is_boolean(x)
      return x == true or x == false
    end

### Q6. 表达式 `(x and y and (not z) or ((not y) and x)` 中的括号是否必要？你会推荐在这个表达式中用括号吗？

因为运算符的优先级，`not` 高于 `and` 高于 `or`，所以上述表达式中的括号不是必需的。但是考虑到明确性和可读性，我推荐使用括号。

### Q7. 下面脚本的输出是什么？请解释原因？

    sunday = "monday"; monday = "sunday"
    t = {sunday = "monday", [sunday] = monday}
    print(t.sunday, t[sunday], t[t.sunday])

上述脚本的输出是：

    monday   sunday    sunday

原因：表 `t = {sunday = "monday", [sunday] = monday}` 等价于 `t = {["sunday"] = "monday", ["monday"] = "sunday"}`，而输出语句 `print(t.sunday, t[sunday], t[t.sunday])` 相当于 `print(t["sunday"], t["monday"], t["monday"])`。

### Q8. 假设你想要创建一个表，其中每一项是一个转义字符及其对应的含义。你该使用怎样的表构造器？

常用的数组构造器和记录构造器，都不能把转义字符作为索引，所以我选择通用构造器：

    escnames = {
      ["\a"] = "bell", ["\b"] = "back space",
      ["\f"] = "form feed", ["\n"] = "newline",
      ["\r"] = "carriage return", ["\t"] = "horizontal tab",
      ["\v"] = "vertical tab", ["\\"] = "backslash",
      ["\""] = "double quote", ["\'"] = "single quote",
    }
