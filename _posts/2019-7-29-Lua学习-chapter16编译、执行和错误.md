---
layout:     post
title:      Lua 学习 chapter16
subtitle:   chapter16
date:       2019-7-29
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 编译
2. 错误
3. 错误信息和栈回溯


> You must try your best. Then you will have a good improvement.

## 编译
dofile函数，加载文件并且执行文件中的代码。其实dofile并不是运行lua代码的核心，核心是loadfile函数。

```lua
function dofile(filename)
	local f = assert(loadfile(filename))
	return f()
end
```

如果loadfile执行失败，那么函数assert会引发一个错误。相较于dofile，loadfile在发生错的时候会返回nil以及错误信息。而且在多次运行这个文件的时候，loadfile只需要加载一次后面可以多次调用。

函数load函数和loadfile函数非常的相似，但是加载的不是文件而是字符串。函数load总是在全局环境中编译代码片段，所以里面的值也都是针对于全局变量的。

* loadfile() 只加载编译代码，不执行, 返回一个function. loadfile的时候必须使用文件全名（带上后缀）
* dofile() 加载执行代码，每调用dofile一次，都会重新编译执行一次。
* require() 只执行一次，会保存已加载过的文件，不会重复加载执行。（常用）加载文件时，require会在packeage.loader中查找模块是否存在，若存在直接返回，否则，加载模块文件。不用加上文件后缀名。

loadfile的思考：

执行下面的代码，为什么会报错呢？这有点类似于只是把代码段里面的程序代码换掉了，但是这些全局变量并没有被执行过，当然找不到了，所以这里只是更换代码，要想代码生效还需要执行一下。

在require的时候我们发现代码被执行了，而不是简单的loadfile，所以我们推测应该是dofile而不是loadfile。

```lua
--local test = require("moduleTest")
local t = loadfile("moduleTest.lua")
--t()
Pack.Print()

--lua.exe: hello.lua:4: attempt to index a nil value (global 'Pack')
--stack traceback:
	--hello.lua:4: in main chunk
	--[C]: in ?
```

## 错误

可以通过assert函数来判断一个函数是否成功执行，没有成功执行则会返回错误信息。assert函数第二个参数为可选的错误信息，可以输出你想要输出的错误信息。

通过assert函数抛出错误信息可以让函数继续执行。

如果想要在lua代码中处理错误，那么就应该使用函数pcall来封装代码。函数pcall能够返回传递给error的任意lua语言类型的值。

```lua
local ok, msg = pcall(functionname)
if ok then
	regular code
else
	error-handling code
end
```

函数pcall会以一种保护模式来调用它的第一个参数，以便来捕捉该函数执行中的错误。无论是否发生错误pcall都不会引发错误。

## 错误信息和栈回溯

当遇到内部错误时，lua语言负责产生错误对象，如果错误对象是一个字符串，那么lua语言还会尝试把一些有关错误发生的位置信息附上：

```lua
local status,err = pcall(function() error("my error")end)
print(err) -->stdin:i:my error
```