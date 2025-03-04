---
layout:     post
title:      Lua 学习 chapter1
subtitle:   chapter1
date:       2019-4-21
author:     Jow
header-img: img/in-post/luaStudy/chapter.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 程序段
2. 一些词法的规范
3. 全局变量
4. 类型的值

> Sometimes, you don't need to ask why, what you should do is just to do it.
> Sometimes, you should know what you shouldn't do this, because this reminds you whenever you just don't do it.
>[https://joedf.ahkscript.org/LuaBuilds/](https://joedf.ahkscript.org/LuaBuilds/) 已经编译好的lua，适用于windows

## 程序段

我们将Lua语言执行的每一段代码称为程序段，即一组命令或表达式组成的序列。

## 一些词法规范

Lua的标识符可以由任意字母、数字和下划线组成，但是不能以数字开头。

注释相关

--[[

]]

这个是多行注释

可以使用这样的小技巧

--[[

--]]

这样注释

---[[

--]

取消多行注释

## 全局变量

在lua语言中，全局变量无需声明即可使用，使用未经初始化的全局变量也不会导致报错。使用未经初始化的全局变量时，得到的结果是nil。

**当把nil赋值给全局变量时，lua回收该全局变量**

## 类型的值

Lua的语言是一种动态类型的语言，在这种语言中没有类型的定义，每个值带有其自身的类型信息。

Lua拥有8种基本类型

1. type(nil) --> nil
2. type(true) --> boolean
3. type(10)  --> number
4. type(io.stdin) --> userdata
5. type(print) --> function
6. type(type) --> thread
7. type({}) --> table
8. type(type(X)) --> string

**type函数的返回值永远都是string**

### nil

nil是只有一个nil值的类型，它的主要作用，是与其他所有的值进行区分，全局变量未被赋值的初始值就是nil，当一个变量被赋值为nil这个变量所引用的值就会被释放。

### boolean

Boolean拥有两个值，分别是false and true。

但是在条件检测中，Boolean值并非是条件检测的唯一方式，任何值都可以用于条件检测。

在lua中，除了**false和nil其它值都会被认为是真值。**

条件的判断的返回值：

在lua中，逻辑运算符有：and、or以及not。使用任意一种逻辑运算符它都会存在返回值。

在使用and的时候，如果第一个值为false，就会返回第一个值，否则返回第二个值。

在使用or的时候，如果第一个值为true，则返回第一个值，否则返回第二个值。

**and 和 or 都遵循短路求值，即当第一个值能返回的时候就不回去算第二个值**

not的返回值永远都是Boolean类型的。

## 独立解释器 ##

即是一个可以直接运行Lua语言的小程序，即控制台输入语句直接执行。