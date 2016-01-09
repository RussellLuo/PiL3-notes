# 9. 协程（Coroutines）

一个 `协程` 类似于多线程概念中的一个线程：它是一个执行序列，有自己的堆栈、自己的局部变量、自己的指令指针；但与此同时，它又与其他协程共享全局变量和其他大部分东西。

线程与协程的主要区别在于：

区别       | 线程                                             | 协程
---------- | ------------------------------------------------ | --------------------------------------------------------------
**并行性** | 在多核机器上，多个线程（理论上）可并行执行       | 同一时间，只能有一个协程处于执行状态
**协作性** | 线程之间没有协作：线程的切换，由操作系统进行调度 | 协程之间相互协作：协程的切换，需要由执行中的协程主动交出执行权


## 1. 协程基础（Coroutine Basics）

Lua 把所有协程相关的函数都打包到了表 `coroutine` 中：

函数                | 含义
------------------- | ------------------------
`coroutine.create`  | 创建一个协程
`coroutine.consume` | （重新）开始执行一个协程
`coroutine.yield`   | 暂停执行一个协程
`coroutine.status`  | 获取协程的执行状态

一个简单的示例：

    co = coroutine.create(function()
      for i = 1, 2 do
        print("co", i)
        coroutine.yield()
      end
    end)

    # 协程的类型是`thread`，创建后的初始状态是`suspended`
    print(co)                    --> thread: 0x77b380
    print(coroutine.status(co))  --> suspended

    # 第一次执行，打印后暂停
    coroutine.resume(co)         --> 打印：co    1；返回：true
    print(coroutine.status(co))  --> suspended

    # 第二次执行，打印后暂停
    coroutine.resume(co)         --> 打印：co    2；返回：true
    print(coroutine.status(co))  --> suspended

    # 第三次执行，协程执行结束
    coroutine.resume(co)         --> 没有打印；返回：true
    print(coroutine.status(co))  --> dead

    # 已经结束的协程，不能再执行
    coroutine.resume(co)         --> 没有打印；返回：false  cannot resume dead coroutine
    print(coroutine.status(co))  --> dead

协程的执行状态，除了 `suspended`、`running` 和 `dead` 外，还有 `normal`。如果一个协程正在执行另一个协程，此时，我们称第一个协程处于 `normal` 状态。

协程的参数和返回值的处理规则：

1. `resume` 的额外参数会传递给协程主函数

        co = coroutine.create(function(a, b, c)
          print(a, b, c + 2)
        end)
        coroutine.resume(co, 1, 2, 3)  --> 打印：1    2   5

2. 如果协程主函数中有 `yield` 调用，则传递给 `yield` 函数的参数就是 `resume` 的返回值

        co = coroutine.create(function(a, b)
          coroutine.yield(a + b, a - b)
        end)
        coroutine.resume(co, 20, 10)  --> 返回：true    30    10

3. 如果协程主函数中有 `yield` 调用，则传递给 `resume` 函数的额外参数就是 `yield` 的返回值

        co = coroutine.create(function(a, b)
          print("main", a, b)
          print("yield", coroutine.yield())
        end)
        coroutine.resume(co, 1, 2)  --> 打印：main     1   2
        coroutine.resume(co, 1, 2)  --> 打印：yield    1   2

4. 如果协程主函数有 `return` 语句，则 `return` 的值也是 `resume` 函数的返回值

        co = coroutine.create(function()
          return 6, 7
        end)
        coroutine.resume(co)  --> 返回：true     6   7


## 2. 管道与过滤器（Pipes and Filters）

暂略


## 3. 用协程实现迭代器（Coroutines as Iterators）

使用协程时，Lua 中有一种惯用模式：创建一个函数，该函数会在内部创建一个协程，并且会返回另一个用于执行该协程的函数。例如：

    function permutations(a)
      local co = coroutine.create(function() permgen(a) end)
      return function()
        local code, res = coroutine.resume(co)
        return res
      end
    end

Lua 还专门为此提供了一个便利函数 `coroutine.wrap`。借助于 `coroutine.wrap` 函数，我们可以更轻松地实现上述函数：

    function permutations(a)
      return coroutine.wrap(function() permgen(a) end)
    end

与 `coroutine.create` 类似，`coroutine.wrap` 也会创建一个协程，但是二者的区别在于：

区别     | `coroutine.create`                      | `coroutine.wrap`
-------- | --------------------------------------- | ----------------------------------------------
返回值   | 返回一个协程                            | 返回一个函数，调用该函数会触发执行其内部的协程
错误处理 | 执行协程出错时，`resume` 会返回错误信息 | 调用函数出错时，会直接抛出错误
灵活性   | 高                                      | 低


## 4. 非抢占式的多线程（Non-Preemptive Multithreading）

与多线程不同，协程之间是非抢占式的：协程的执行权切换是主动的、明确的。因此，借助协程，可以实现非抢占式的多线程。但这种非抢占性也有它的缺点：如果一个协程的执行被阻塞，整个程序也会随之被阻塞。这个问题的解决办法是：将阻塞操作转换为非阻塞操作。

例如，如果我们想要下载多个网页，使用协程可以很轻易地实现并发下载。以下是实现该功能的全部代码：

    local socket = require "socket"

    -- Download a Web page
    function download(host, file)
      local c = assert(socket.connect(host, 80))
      local count = 0  -- counts the number of bytes read
      c:send("GET " .. file ..  " HTTP/1.0\r\n\r\n")
      while true do
        local s, status = receive(c)
        count = count + #s
        if status == "closed" then break end
      end
      c:close()
      print(file, count)
    end

    -- Receive the file stream of the Web page
    function receive(connection)
      connection:settimeout(0)  -- do not block
      local s, status, partial = connection:receive(2^10)  -- in blocks of 1KB
      if status == "timeout" then
        coroutine.yield(connection)  -- yield non-false value to indicate unfinished
      end
      return s or partial, status
    end

    -- The dispatcher
    threads = {}  -- list of all live threads

    function get()
      -- create coroutine
      local co = coroutine.create(function()
        download(host, file)
      end)
      -- insert it in the list
      table.insert(threads, co)
    end

    function dispatch()
      local i = 1
      local timedout = {}
      while true do
        if threads[i] == nil then  -- no more threads
          if threads[1] == nil then break end
          i = 1
          timedout = {}
        end
        local status, res = coroutine.resume(threads[i])
        if not res then  -- thread finished its task?
          table.remove(threads, i)
        else
          i = i + 1
          timedout[#timedout + 1] = res
          if #timedout == #threads then  -- all threads blocked?
            socket.select(timedout)  -- wait for any connection to change status
          end
        end
      end
    end

上述程序的使用方式如下：

    host = "www.w3.org"

    get(host, "/TR/html401/html40.txt")
    get(host, "/TR/2002/REC-xhtml1-20020801/xhtml1.pdf")
    get(host, "/TR/REC-html32.html")
    get(host, "/TR/2000/REC-DOM-Level-2-Core-20001113/DOM2-Core.txt")

    dispatch()  -- main loop


## 练习题（Exercises）

暂略
