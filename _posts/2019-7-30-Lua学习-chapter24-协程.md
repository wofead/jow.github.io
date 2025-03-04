---
layout:     post
title:      Lua 学习 chapter24
subtitle:   chapter24
date:       2019-7-30
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 协程
2. yield

> 人人真真的生活过，学习过，改变过，努力过，才能创造出一个满意的自己。

## 协程

> 协程是一系列的可执行语句，拥有自己的**栈、局部变量和指令指针**，同时协程又与其他协程共享了全局变量和其他几乎一切资源。线程和协程的主要区别在于，一个**线程程序可以并行运行多个线程**，而**协程却需要彼此协作的运行，即任意指定时刻只能有一个协程运行**。

协程相关的函数都在**coroutine**表中，其中create为创建协程函数，参数为协程要执行的代码函数（协程体）。它会返回一个thread类型的协程。

一个协程拥有四种状态：挂起，运行，正常和死亡。

当一个协程被创建的时候，它处于挂起状态。函数resume用于启动或再次启动一个协程的运行，并将其状态由挂起改为运行。协程函数（协程体）执行完毕之后进入死亡状态。当使用协程A唤醒协程B的时候，协程A即不是挂起状态（因为不能唤醒协程A），也不是运行状态（因为B正在运行）。所以A此时的状态就被称为正常状态。

lua语言一个非常有用的机制是通过一对resume-yield来交换数据，**第一个resume函数**（没有对应等待它的yield）会把所有额外参数传递给协程的**主函数**。

在函数resume的返回之中，第一个返回值为true时表示没有错误，之后的返回值对应函数yield的参数。

可以将唤醒协程的函数放在一个函数中，因为这种模式比较常见，所以lua语言专门提供了一个特殊的函数coroutine函数来完成这个功能。

当执行wrap中的函数时候，就唤醒协程。

```lua
co = coroutine.create(function(a, b, c)
    return a + b + c
end)

print(coroutine.resume(co, 1, 2, 3)) --> true 6。执行状态和结果

co = coroutine.create(function(a, b, c)
    print(coroutine.yield(a + b, b + c)) --2 4 6,返回值是唤醒这个yield的参数值
    return a + b + c
end)

print(coroutine.resume(co, 1, 2, 3)) -- true	3	5,返回yield的参数，是最开始传进来的参数
print(coroutine.resume(co, 2, 4, 6)) -- true	6，返回结果，根据最开始传进来的参数
print(coroutine.resume(co, 2, 4, 6)) --false	cannot resume dead coroutine

function permutations(a)
	return coroutine.wrap(function() permgen(a) end)
end
```
生产者与消费者的例子：
```lua
function receive(prod)
    local status, value = coroutine.resume(prod)
    return value
end

function send(x)
    coroutine.yield(x)
end

function producer()
    return coroutine.create(function()
        while (true) do
            local x = io.read()
            send(x)
        end
    end)
end

function filter(prod)
    return coroutine.create(function()
        for line = 1, math.huge do
            local x = receive(prod)
            x = string.format("%5d %s", line, x)
            send(x)
        end
    end)
end

function consumer(prod)
    while true do
        local x = receive(prod)
        io.write(x, "\n")
        local exit = string.match(x, "%l+")
        if tostring(exit) == "q" then
            break
        end
    end
end

consumer(filter(producer()))
```

## yield

一般我们沉睡一个协程的时候都是使用yield，但是yield又没有参数，是怎么确定沉睡的那个协程的呢？

在这里我首先测试了一下如下的代码，发现会报错，突然明白，原来协程的阻塞 必须写到参数的函数里面，才能阻塞这个协程，写到主线程里面，因为知不道要阻塞的协程会直接报错。

所以下面的代码应该将 coroutine.yield()放到test函数里面去，就正确了。
```lua
local function test()
    print("test")
	--coroutine.yield()
    print("test1")
end

local co = coroutine.create(test)
coroutine.yield()
```

	lua.exe: attempt to yield from outside a coroutine
	stack traceback:
	[C]: in function 'coroutine.yield'






