---
layout:     post
title:      Lua 学习 chapter10
subtitle:   chapter10
date:       2019-6-10
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 正则表达式
2. 模式匹配相关的函数
3. URL编码
4. 制表符的展开
5. 诀窍
6. 模式


> Just be handsome.

## 正则表达式
1. . 任意字符
2. %a 字母
3. %c 控制字符
4. %d 数字
5. %g 除空格外的可打印字符
6. %l 小写字母
7. %p标点符号
8. %s 空白字符
9. %u 大写字母
10. %w 字母和数字
11. %x 十六进制数字

%它们的大写字母表示它们的补集。

还有一些魔法字符，也可以称之为元字符。

其中与其他的语言不同的是在lua中%是转义字符而不是'\'。

1. \+ 重复一次或多次
2. \* 重复零次或者多次（最大匹配）
3. \- 重复零次或者多次（最小匹配）位于[]中中间表示范围
4. ？ 可选
5. ^ 位于模式的开头 从头开始匹配  位于[]中开头表示非
6. $ 位于模式的结尾 从尾部向前匹配

在lua中可以使用()来对匹配到的结果进行返回。这个称之为**捕获**。


```lua
pair = "name = Anna"
key, value = string.match(pair, "(%a+)%s*=%s*(%a+)")
print(key, value) -- name Anna

s = [[then he said: "it's al right"!]]
q, quotedPart = string.match(s, "([\"'](.-)%1")
print(quotedPart) -- it's al right

--find的使用
string.find("a [word]","[")
-- stdin:1: malformed pattern(missing ']')
string.find("a [word]","[",1,true)
```

## 模式匹配相关的函数
1. string.find(ste,reg) --返回开始和结束的index，find是存在是个参数的，如果寻找的是特殊的字符，例如‘[’是模式匹配中的，那就需要第三个参数和第四个参数，第三个参数表明从哪个位置开始索引，第四个数字表示简单查询。
2. string.match -- 返回匹配到的内容
3. string.gmatch --返回一个函数，通过函数可以遍历一个字符串出现的指定模式
4. string.sub --截取字符串
5. string.gsub --替换字符串，当第三个参数是字符串的时候，直接替换。如果是函数，会把匹配的内容作为参数传给函数，将函数的返回值替换原来的值，如果是表，匹配的内容作为键值去取数据。

string.sub 有三个参数，它的第三个参数不仅仅可以是字符串还可以是函数或者表。

当时函数的时候每次匹配到合适的字符串的时候就调用函数参数为匹配到的值，用函数的返回值来替换字符串；当是表的时候，将匹配到的字符串作为key值来得到用来替换的字符串。

%f[char-set]表示前置模式。该模式只有在后一个字符位于char-set内而前一个字符不在时匹配一个空字符串。


```lua
s = "some string"
local words = {}
for w in string.gmatch(s, "%a+") do
    print(w)
    table.insert(words, w)
end -- some string 

name = "Lua"
status = "great"
print(expend("$name is $status")) --Lua is great	2 其中2表示匹配成功的字符串

--使用%bxy，x和y为任意字符，匹配它们两个中间的字符串
local s = "a (enclosed (in) parentheses) line"
local smatch = string.match(s, "%b()")
print(smatch)
```
## url编码

这种编码方式会将特殊字符编码为"%xx"的形式（=、&、+）,会将空格转换为加号"+"。例如字符串"a+b = c",就会别编码成"a%2Bb+%3D+c"。

```lua
function unescape(s)
	s = string.gsub(s, "+", " ")
	s = string.gsub(s, "%%(%x%x"),function(h)
		return string.char(tonumber(h, 16)
	end)
end
```

## 制表符的展开

在lua语言中，像"()"这样的空白捕捉具有特殊的含义，表示捕捉模式在目标字符串中的位置。

```lua
print*string.match("hell0", "()ll()") -->3  5
--输出的是匹配的第一个字符的位置和输出最后一个字符之后的那个字符的位置
function expandTabs(s, tab)
	tab = tab or 8
	local corr = 0
	s = string.gsub(s, "()\t",function(p)
		local sp = tab - (p - 1 + corr) % tab
		corr = corr - 1 + sp
		return string.rep(" ", sp)
	end)
return s
```

## 诀窍

1. "(.-)%$"  和 "^(.-)%$"  --前面一种模式的匹配会导致当没有'$'符号时，模式匹配算法会从字符串的第一个位置开始匹配，到尾之后从第二个位置再次匹配，算法的复杂度为n的二次方。
2. "%a*" 会导致空字符串


所以在使用 ".-"的时候注意会匹配到空字符串。

## 模式

%bxy匹配成对的字符串和%f[char-set]的前置模式（后一个字符位于模式内，而前一个字符不在模式内）

```lua
local s = "a [word]!"

local s = "a [word]!"

print(string.match(s,"()(%b[])()")) --3 [word] 9，()用来捕获输出值，当使用捕获的时候，没有输出捕获的内容，在这里双括号里面没有任何东西的时候，会输出匹配内容的第一个字符的位置和最后一个字符的后一个位置。

print(string.sub(s,string.match(s,"()%b[]()"))) --[word]!

s = "one day i will find one way!"

print(string.gmatch(s,"%f[%w][%w*]%f[%W]"))

for word in string.gmatch(s,"%f[%w][%w*]%f[%W]") do
    print(word)
end

-- 只有一个i
-- 如果将[%w*]换成one，将会输出两个one。
```


在捕获中，还有一个非常重要的概念，那就是匹配第几次捕捉到的内容，例如匹配双引号或者单引号中的字符串，如果仅仅使用"[\"'].-[\"']",这样的模式，但是遇到"it's all right!"这样的就会出错。

所以在这里我们通过通过捕捉，然后使用“%n”这样的形式(表示匹配第n个不会的副本）。

还可以用在gsub中：

```lua
s = [[then he said:"it's all right!"]]
print(string.match(s,"([\"'])(.-)%1")) --捕获第一个匹配值，和要匹配的内容，其中%1里的内容应该是'"'

s = "hello world!"

print(string.gsub(s,"(.)(.)","%2%1")) --ehll oowlr!d	6，6表示替换发生了几次
```

