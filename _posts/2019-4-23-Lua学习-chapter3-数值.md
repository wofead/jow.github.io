---
layout:     post
title:      Lua 学习 chapter3 数值
subtitle:   chapter3：数值
date:       2019-4-23
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 算术计算
2. 关系运算
3. 数学库
4. 表示范围
5. 运算符的优先级
6. 练习

> To be honest, i think what you have compare to others expection has a long way.

## 算术计算

在lua中，除法运算操作的永远是浮点数而且产生的结果也是浮点数。

在新的版本中针对于整数除法引入了一个floor除法的新运算符‘//’，floor除法会对得到的上进行向下取整，从而保证结果是一个整数。

如果操作数是整形类型结果就是整型类型，如果操作数是浮点类型那么结果就是浮点类型（其值是个整数），类似的取模运算(%)也是一样的。

## 关系运算

Lua提供了下列的关系运算：

< > <= >= == ~=  这个关系运算的返回值都是Boolean类型的。

## 数学库

Lua提供了标准的数学库 math，包含三角函数，指数，取整，最大最小，用于生成伪随机的random函数以及pi和huge(最大可表示的值)。

### 随机数发生器

lua中随机数共有三种调用方式：

1. 不带参：随机返回一个伪随机的[0,1)的数
2. 带一个整型参：返回一个[1,n]的伪随机整数
3. 带两个整型参：返回一个[n,m]的伪随机整数

函数randomseed用于设置伪随机数发生器的种子，该函数的唯一参数就是数值类型的种子。

在一个程序启动时，系统固定使用1为随机数发生器。如果不设置其它种子，每次系统启动都会产生一样的随机数。

所以为了解决这个问题通常使用**math.randomseed(os.time())** 来作为种子。

### 取整函数
1. floor：向下取整
2. ceil：向上取整
3. modf：向0取整

就近取整：

```lua
function round(x)
	local f = math.floor(x)
	if (f == x) or (x % 2 == 0.5)  then 
		return x
	else 
		return math.floor(x + 0.5)
	end
end
```

## 表示范围

lua使用64个比特位来存储整型值，其最大值为2^63 - 1。

我们可以简单的使用整数加上0.0来转换成浮点数。

在lua的管理中，超过2^53之后这样转换就会失去精度。

特别是在进行浮点数的运算过程中，可能会导致计算的不精确。例如：

12.7 -20 + 7.3，即便使用双精度来表示结果也不是0，因为12.7和7.3的二进制表示不是有限的小数。

## 运算符的优先级

1. ^
2. 一元运算符 （- # ~ not）
3. /  //  %  （//  floor除）
4. + -
5. ..（连接）
6. << >> （按位移位）
7.  & 
8.  ~
9.  |
10.  < > <= >= ~= ==
11.  and or

## 练习      
                                                                                                                                                   
乘2，就是左移一位。


使用math.random生成正太分布的为随机数发生器

采用 Box-Muller 变换可以将均匀分布转化成正态分布

Z=R*cosθ 或 Z=R*sinθ （其中 R = sqrt(-2*ln(u1)), θ = 2*π*u2，u1和u2是（0，1]的均匀分布随机数，得到的Z就是标准正态分布随机数。

1. 生成均匀分布的随机数，math.random 函数生成的是 [ 0,1 ) 之间的随机数，只需在生成 0 时改为生成 1 。

2. 通过Box-Muller公式得到服从正态分布的随机数

```lua
local rand1,rand2=math.random(),math.random()
  rand1=rand1==0 and (1) or (rand1)
  rand2=(rand2==0) and (1) or (rand2)
  return math.sqrt(-2.0*(math.log(rand1)))*math.sin(2*math.pi*rand2);	
```
