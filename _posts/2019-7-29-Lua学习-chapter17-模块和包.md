---
layout:     post
title:      Lua 学习 chapter17
subtitle:   chapter17
date:       2019-7-29
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. require函数
2. 模块
3. 搜索路径
4. 搜索器
5. 子模块和包


> Continue, come on.

## require函数

ruquire函数可以加载任意模块，然后创建和返回一个表.

```lua
local mod = require "mod"
mod.foo()

local m = require "mod"
local f1 = m.foo
f1()

local f2 = require "mod".foo
f2()
```
require 函数在表**package.loaded**中检查模块是否已经被加载过。如果加载过就返回相应的值，这就避免的重复的加载。没有加载，就会通过loadfile来对其进行加载。关于文件的搜索目录，是由变量package.path执行。loadfile其实是一个加载器(loader)函数（加载器就是一个被调用时加载木块的函数）。在Xlua中，我们可以通过定制Loader，让它符合自己的要求，也可以使用内置的Loader。

当然require也可以加载c标准库，则使用package.loadlib进行加载，这个底层函数会查找名为luaopen_modname的函数。

如果没有lua文件，就回去加载c标准库，使用底层函数package.loadlib进行加载。

如果加载函数有返回值，那么函数require会返回合格值，将其保存在packag.loaded中，如果没有返回值，且package.loaded[@rep{moduname}]为空，函数require就假设该模块的返回值是true。如果不存在这种，补偿会造成重复加载。

要强制加载同一模块两次，可以先将模块从package.loaded中删除：package.loaded.modname = nil.

对require的测试：

首先我们在moduleTest.lua中声明全局的一个table，但是并不返回，这个时候我们发现package.loaded表中存了moduleTset的键，值为true，然后我们取消注释return的哪一行，发现loaded表中存的就是Pack这张全局表的内存地址了。

require是存在返回值的，如果是复杂的类型，会直接返回这个复杂类型的地址。

```lua
--moduleTest.lua
Pack = {}
function Pack.Print()
    print("lua 好恶心！")
end
Pack.Print()
--return Pack


--main.lua
local test = require("moduleTest")
Pack.Print()

local pa = package.loaded
print(pa)
```


## 模块

在lua中，简单的使用模块的方式就是使用表将所有属性放到这个表中（可以封装成类），然后最后返回这个表。

如果不想返回的话，可以以选择给package.loaded[M] = M。赋值的方法。

模块是可以重新命名的，直接将返回的值赋值给另一个就行了。

## 搜索路径

在搜索一个lua文件时，函数require使用的路径和典型的路径略有不同。典型的路径是很多目录组成的列表，并在其中搜索指定的文件。不过 ISO C（Lua语言依赖的抽象平台）并没有目录的概念。所以，函数require使用的路径是一组模板，其中的每项都制定了模板名（函数require的参数）转换为文件名的方式。更确切的说，这种路径中的每一个模板都是一个包含可选问好的文件名。对于每一个模板，函数require会用模板名来替换每一个问号，然后检查是否存在对应的文件。路径中的模板以在大多数操作系统中很少用于文件名的分号隔开。

package.searchpath中实现了搜索库的所有规则，该函数的参数包括模板名和路径，然后遵循上述规则来搜索文件。


## 搜索器

在现实中，require比此前描述过得稍微复杂一些，搜索lua文件和c标准库的方式只是更加通用的搜索器的两个实例。一个搜索器是以一个以模块名为参数，以对应的模块的加载器或nil(找不到)为返回值的简单函数。

预加载（preload)搜索器使得我们能够为要加载的木块定义任意的加载函数。预加载搜索器使用一个名为package.preload的表来映射模块名称和加载函数。当搜索执行的木块名时，改搜索器只是简单地在表中搜索指定的名称。如果他找到了对应的函数，那么就将该函数作为相应模块的加载函数返回，否则，返回nil。

## 子模块和包

lua支持具有层次结构的模块名，通过点来分隔名称中的层次。例如，一个名为mod.sub的模块是模块mod的一个子模块。一个包是一颗由模块组成的完整的树，它是lua语言中用于发行程序的单位。

当加载一个名为mod.sub的模块的时候，函数require依次使用原始的模块名“mod.sub”作为键来查询表package.loaded和package.preload。这里，模块中的点像模块名中的其他字符一样，没有特殊含义。


