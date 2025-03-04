---
layout:     post
title:      Lua 学习 chapter5
subtitle:   chapter5
date:       2019-4-25
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 表索引
2. 表构造器
3. 数组、列表和序列
4. 遍历表
5. 安全访问
6. 表标准库
7. 练习

> Work hard, come on baby.
> This chapter expresses the table.

## 表索引
a.x 表示的意思和a["x"]一样，所以前者的可读性更高。
```lua
local a = {}  --空白表，建立了一个表的索引
a.x = 10
a.x --> 10
a.y --> nil
a = nil -->取消对表的索引，然后lua的垃圾处理器就会对其进行回收
```

## 表构造器
表构造器是用来创建和初始化表的表达式，也是lua语言中独有的也是最有用、最灵活的机制之一。

在构造表的时候可以使用记录式和列表式，当然你也可以混用它们：

```lua
days= {"Mon","Tues","Wednes","Thurs","Fri","Satur","Sun"}
a = {x = 10, y = 100}
poly = {
	color = "blue",
	thickness = 2,
	{x = 0, y = 0}, --polyline[1]
	nppoints = 4,
	{x =2, y =2}, --polyline[2]
	{x = 5, y= 6}, --polyline[3]
	[4] = {x = 7, y =8},
	[6] = {x= 10, y = 10}
}
```

可以看到在表中你可以使用键值对方式，当然也可以直接存入值(或者整数作为元素的索引)。直接存入值的地方就可以通过整数来索引像上面的表索引的那样。

使用ipairs循环打印的时候，使用整数索引的值能够被打印出来，正如顺序构造的那样。

![](https://i.imgur.com/MMD6oy1.png)

当分别使用ipairs和pairs进行循环遍历的时候，我们会发现其中的特点。这说明表的存储结构，它是由数组部分和hashtable部分组成的，在输出的时候数组的部分会被优先输出，之后是hashtable部分。

其中ipairs只会遍历数组部分，而pairs会遍历所有部分。

**ipairs遍历的时候只会从1开始，直到顺序断开，pairs的遍历是乱序的，不能保证的**

当然你也可以使用特定的数字来作为表的索引，需要使用中括号,如上例[4]就能被ipairs遍历到，但是[6]就不行，因为不连续。

## 数组、列表和序列

在table中，想表示常见的数组或者列表，那么只需要使用整形作为索引的表即可。

当然当列表中不存在nil的时候这样的数组可以被称之为序列。

更准确的说序列是由指定的n个正整数为键值组成的[1...n]的集合。在中间既不能存在空洞(值为nil或者干脆没有键值直接越到后面的整数去)，这样通过**#访问数组的长度只会返回空洞之前的有效长度。**

## 遍历表
1. ipairs 针对于序列，顺序输出
2. pairs  针对于所有表，输出所有元素，但是不保证顺序(因为存储方式为hashtable)
3. for i = 1, #t do print(t[i]) end  使用for循环

## 安全访问

可以通过**or**来实现安全访问：

```lua
E = {}
zip = (((company or E).director or E).address or {}).zipcode 
```

这样不会报错，最后返回的值也是nil，而且还减少了对表的访问。

## 表标准库

1. table.insert(t,pos,val)  -->向指定的位置插入一个值，当然也可以不指定位置，就会直接在最后插入
2. table.remove(t,pos) --> 移除指定位置的元素，如不指定，移除最后一位，移除之后后面的元素会自动前移
3. table.move(a,f,e,t) --> 将表a中，从f到e的元素移到从t开始的位置
4. table.move(a,f,e,t,{}) --> 将a表中从f到e的元素移动到新表{},从t开始的位置
5. table.concat("hello"," world") --> hello world 连接字符串

```lua
local t2 = { 1, 2, 3, 4, 5, 6, 7 }
local t3 = {}
table.insert(t3, 1)
table.move(t2, 1, #t2, 2)
table.move(t2, 1, #t2, 2, t3)
for i, v in ipairs(t2) do
    print(t2[i])
end

for i, v in ipairs(t3) do
    print(t3[i])
end
```

![](https://i.imgur.com/q882Pfl.png)

## 练习

```lua
a={}
a.a = a
print(a.a.a)
a.a.a.a = 3
print(a.a.a)
```

在赋值3之前无论打印那个a他们都指向的是那张空表，当赋值之后a.a 指向3，所以a.a.a即3.a会报错



