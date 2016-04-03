# 11. 数据结构（Data Structures）

在 Lua 中，表不是数据结构的一种，而是数据结构的全部。所有其他语言中常见的数据结构，如数组、记录、列表、队列、集合等等，都可以用表来实现。


## 1. 数组（Arrays）

    -- 使用构造器创建并初始化
    a = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

    -- 或者，先创建空表再添加元素
    a = {}
    for i = 1, 10 do
      a[i] = 0
    end

Lua 语言的惯例是：数组索引从 1 开始计。


## 2. 矩阵与多维数组（Matrices and Multi-Dimensional Arrays）

在 Lua 中表示矩阵，有两种方式（以二维数组为例）：

1. 使用数组的数组

        mt = {}       -- 创建矩阵
        for i = 1, N do
          mt[i] = {}  -- 创建新行
          for j = 1, M do
            mt[i][j] = 0
          end
        end

2. 将两个索引合并成一个索引：

        mt = {}
        for i = 1, N do
          for j = 1, M do
            mt[(i - 1) * M + j] = 0
          end
        end

如果以字符串为索引，则可用特殊字符来分隔合并：如 `mt[i .. ":" ..j]`，或者 `mt[i .. "\0" ..j]`。


## 3. 链表（Linked Lists）

空链表：

    list = nil

在链表头插入一个值为 `v` 的元素：

    list = {next = list, value = v}

遍历链表：

    local l = list
    while l do
      <visit l.value>
      l = l.next
    end


## 4. 队列与双端队列（Queues and Double Queues）

暂略


## 练习题（Exercises）

暂略
