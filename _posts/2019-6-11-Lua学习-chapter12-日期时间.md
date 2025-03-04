---
layout:     post
title:      Lua 学习 chapter12
subtitle:   chapter12
date:       2019-6-11
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 日期和时间
2. 日期和时间的处理


> What is the cost of lies? It's not that we'll mistake them for the truth. The real danger is that if we hear enough lies, then we no longer recognize the truth at all. What can we do then? What else is left but to abandon even the hope of truth and content ourselves instaed with stories? In these stories, it doesn't matter who the heroes are. All we want to know is: who is to blame? No friends. Or, at least, not important ones.

## 日期和时间
在lua中日期的表示分为两种方式，一种是一个整数，从1970年以来一共多少秒，另一种是日期表，包含以下重要的字段：

year,month,day,hour,min,sec,wday,yday以及isdat,其中wday是这周的第几天，yday是今年的第几天，isdat是否是夏时令。日期表中不包含时区，程序需要结相应的时区对其正确解析。

os.date("*t", os.time()) 生成一个日期表，同样不带第二个参数也可以获取到当前时间的日期表。

```lua
os.time() -- 当前的时间，秒数
os.time({year = 2019,month = 6, day = 13}) --输出秒数
os.date("*t", 906000490)) --{year = 1998, month = 9, day = 16, yday = 259, wday = 4, hour = 23, min = 48, sec = 10, isdst = false}
--os.date() 在一定程度上是函数os.time()的反函数
t = 906000490
os.time(os.date("*t",t)) == t

```

os.date会将日期格式化为一个字符串

1. %a 星期
2. %A 星期几全称
3. %b 月份简称
4. %B 月份全称
5. %c 日期和时间（09/16/98 23:48:10）
6. %d 一个月中的第几天
7. %H 24小时制小时数
8. %I 12小时制小时数
9. %j 一年中的第几天
10. %m 月份
11. %M 分钟
12. %p am or pm
13. %S 秒数 00-60
14. %w 星期几（0-6） 0 is sunday
15. %W 一年中的第几周
16. %x 日期（09/16/98）
17. %X 时间（23:48:10）
18. %y 两位数年份（88）
19. %Y 完整的年份（1988）
20. %z 时区（例如：-0300）
21. %% 百分号

## 日期和时间的处理

os.difftime用来计算两个时间之间的差值，该函数以秒为单位返回差值。它的参数可以是两个日期表。

os.clock也是获取时间，但是其精度更高，返回值是一个浮点数。

```lua
t = os.date("*t")
t.day = t.day + 40
print(os.date("%Y/%m/%d",os.time(t)) --40天后的时间

```



