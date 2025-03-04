---
layout:     post
title:      Lua和C++的交互
subtitle:   Lua和C++
date:       2019-7-25
author:     Jow
header-img: img/timg.jpg
catalog: 	 true 
tags:
    - Lua
    - C++
---

### 目录
1. Lua堆栈
2. Lua堆栈操作
3. C++调用lua
4. Lua调用C++


> There is something beautiful， just to find it.

## Lua堆栈

要理解Lua和C++的交互，首先要清楚Lua堆栈。简单来说Lua和C++交互是通过一个无处不在的虚拟堆栈来的。

在Lua中，lua堆栈是一个结构，堆栈的索引可以是整数或者负数，+1表示栈底，-1表示栈顶。

存入栈的数据类型包含数值，字符串，指针，table，闭包等。

```c
lua_pushcclosure(L, func, 0) // 创建并压入一个闭包

lua_createtable(L, 0, 0)        // 新建并压入一个表

lua_pushnumber(L, 343)      // 压入一个数字

lua_pushstring(L, “mystr”)   // 压入一个字符串
```

这里要说明的是, 你压入的类型有数值, 字符串, 表和闭包[在c中看来是不同类型的值], 但是最后都是统一用TValue这种数据结构来保存的:), 下面用图简单的说明一下这种数据结构:

![](https://i.imgur.com/SJLBAAT.png)

Tvalue结构对应于lua中的所有数据类型，是一个键值对的结构，这就是lua动态类型的实现，它把值和类型绑在一起，用tt记录value的类型，value也是一个联合结构，由value定义，可以
看到这个联合有四个域，先说明简单的：

* p:可以存一个指针，实际上是lua中的light userdata结构
* n：所有数值存在这里，不管是int还是float
* b:Boolean值
* gc：其它诸如table，thread，closure，string需要内存管理垃圾回收的类型都存在这里。 gc是一个指针，它可以指向的类型由联合体GCObject来定义。

从上面我们可以得到结论：

1. lua中，number，boolean，nil，light userdata可以直接存在栈上元素里的，和垃圾回收没有关系。
2. lua中，string，table，closure，userdata，thread存在栈上的元素里的只是指针，它们在声明周期结束的时候被垃圾回收。


## Lua堆栈操作

因为Lua与C/C++是通过栈来通信，Lua提供了C API对栈进行操作。

```c++
#include <iostream>  
#include <string.h>  
using namespace std;  
   
extern "C"  
{  
    #include "lua.h"  
    #include "lauxlib.h"  
    #include "lualib.h"  
}  
void main()  
{  
    //1.创建一个state 
    lua_State *L = luaL_newstate();   //返回一个指向堆栈的指针
       
    //2.入栈操作  
    lua_pushstring(L, "I am so cool~");   
    lua_pushnumber(L,20);  
   
    //3.取值操作  
    if( lua_isstring(L,1)){             //判断是否可以转为string  
        cout<<lua_tostring(L,1)<<endl;  //转为string并返回  
    }  
    if( lua_isnumber(L,2)){  
        cout<<lua_tonumber(L,2)<<endl;  
    }  
   
    //4.关闭state  
    lua_close(L);  
    return ;  
}

int   lua_gettop (lua_State *L);            //返回栈顶索引（即栈长度）  
void  lua_settop (lua_State *L, int idx);   //ua_settop将栈顶设置为一个指定的位置，即修改栈中元素的数量。
//如果值比原栈顶高，则高的部分nil补足，如果值比原栈低，则原栈高出的部分舍弃。所以可以用lua_settop(0)来清空栈。                
void  lua_pushvalue (lua_State *L, int idx);//将idx索引上的值的副本压入栈顶  
void  lua_remove (lua_State *L, int idx);   //移除idx索引上的值  
void  lua_insert (lua_State *L, int idx);   //弹出栈顶元素，并插入索引idx位置  
void  lua_replace (lua_State *L, int idx);  //弹出栈顶元素，并替换索引idx位置的值

```


## c++调用lua

lua和c通信时有这样一个约定，所有的lua中的值由lua来管理，c++中产生的值lua不知道，**如果你(c/c++)想要什么, 你告诉我(lua), 我来产生, 然后放到栈上, 你只能通过api来操作这个值, 我只管我的世界。**

这里我们写一个hello.lua

```lua
str = "I am so cool"  
tbl = {name = "shun", id = 20114442}  
function add(a,b)  
    return a + b  
end
```

写一个c++文件调用lua：

```c++
#include <iostream>  
#include <string.h>  
using namespace std;  
   
extern "C"  
{  
    #include "lua.h"  
    #include "lauxlib.h"  
    #include "lualib.h"  
}  
void main()  
{  
    //1.创建Lua状态  
    lua_State *L = luaL_newstate();  
    if (L == NULL)  
    {  
        return ;  
    }  
   
    //2.加载Lua文件  
    int bRet = luaL_loadfile(L,"hello.lua");  
    if(bRet)  
    {  
        cout<<"load file error"<<endl;  
        return ;  
    }  
   
    //3.运行Lua文件  参数2：传递参数数量，参数3：返回数量，参数4错误处理函数。
    bRet = lua_pcall(L,0,0,0);  
    if(bRet)  
    {  
        cout<<"pcall error"<<endl;  
        return ;  
    }  
   
    //4.读取变量  
    lua_getglobal(L,"str");  
    string str = lua_tostring(L,-1);  
    cout<<"str = "<<str.c_str()<<endl;        //str = I am so cool~  
   
    //5.读取table  
    lua_getglobal(L,"tbl");   
    lua_getfield(L,-1,"name");  
    str = lua_tostring(L,-1);  
    cout<<"tbl:name = "<<str.c_str()<<endl; //tbl:name = shun  
   
    //6.读取函数  
    lua_getglobal(L, "add");        // 获取函数，压入栈中  
    lua_pushnumber(L, 10);          // 压入第一个参数  
    lua_pushnumber(L, 20);          // 压入第二个参数  
    int iRet= lua_pcall(L, 2, 1, 0);// 调用函数，调用完成以后，会将返回值压入栈中，2表示参数个数，1表示返回结果个数。  
    if (iRet)                       // 调用出错  
    {  
        const char *pErrorMsg = lua_tostring(L, -1);  
        cout << pErrorMsg << endl;  
        lua_close(L);  
        return ;  
    }  
    if (lua_isnumber(L, -1))        //取值输出  
    {  
        double fValue = lua_tonumber(L, -1);  
        cout << "Result is " << fValue << endl;  
    }  
   
    //至此，栈中的情况是：  
    //=================== 栈顶 ===================   
    //  索引  类型      值  
    //   4   int：      30   
    //   3   string：   shun   
    //   2   table:     tbl  
    //   1   string:    I am so cool~  
    //=================== 栈底 ===================   
   
    //7.关闭state  
    lua_close(L);  
    return ;  
}
// 将需要设置的值设置到栈中  ,修改值
lua_pushstring(L, "我是一个大帅锅～");  
// 将这个值设置到table中（此时tbl在栈的位置为2）  
lua_setfield(L, 2, "name");
// 创建一个新的table，并压入栈  
lua_newtable(L);  
// 往table中设置值  
lua_pushstring(L, "Give me a girl friend !"); //将值压入栈  
lua_setfield(L, -2, "str"); //将值设置到table中，并将Give me a girl friend 出栈
```
**需要注意的是：堆栈操作是基于栈顶的，就是说它只会去操作栈顶的值。**

函数调用流程是先将函数入栈，参数入栈，然后用lua_pcall调用函数，此时栈顶为参数，栈底为函数，所以栈过程大致会是：参数出栈->保存参数->参数出栈->保存参数->函数出栈->调用函数->返回结果入栈。

类似的还有lua_setfield，设置一个表的值，肯定要先将值出栈，保存，再去找表的位置。

```c++
lua_getglobal(L, "add");        // 获取函数，压入栈中  
lua_pushnumber(L, 10);          // 压入第一个参数  
lua_pushnumber(L, 20);          // 压入第二个参数  
int iRet= lua_pcall(L, 2, 1, 0);// 将2个参数出栈，函数出栈，压入函数返回结果  
lua_pushstring(L, "我是一个大帅锅～");  //   
lua_setfield(L, 2, "name");             // 会将"我是一个大帅锅～"出栈
```

lua_getglobal(L,"var")会执行两步操作：1.将var放入栈中，2.由Lua去寻找变量var的值，并将变量var的值返回栈顶（替换var）。

lua_getfield(L,-1,"name")的作用等价于 lua_pushstring(L,"name") + lua_gettable(L,-2),类似的，会将获得的值替换name。

![](https://i.imgur.com/N0PZZsz.png)

可以看出来, lua中提供的一些类型和c中是对应的, 也提供一些c中没有的类型. 其中有一些药特别的说明一下:

nil值, c中没有对应, 但是可以通过lua_pushnil向lua中压入一个nil值。

**注意: lua_push*族函数都有"创建一个类型的值并压入"的语义, 因为lua中所有的变量都是lua中创建并保存的**, 对于那些和c中有对应关系的lua类型, lua会通过api传来的附加参数, 创建出对应类型的lua变量放在栈顶, 对于c中没有对应类型的lua类型, lua直接创建出对应变量放在栈顶.

例如:   
* lua_pushstring(L, “string”) lua根据"string"创建一个 TString obj, 绑定到新分配的栈顶元素上

* lua_pushcclosure(L,func, 0) lua根据func创建一个 Closure obj, 绑定到新分配的栈顶元素上

* lua_pushnumber(L,5) lua直接修改新分配的栈顶元素, 将5赋值到对应的域

* lua_createtable(L,0, 0)lua创建一个Tabke obj, 绑定到新分配的栈顶元素上

 总之, 这是一个 c value –> lua value的流向, 不管是想把一个简单的5放入lua的世界, 还是创建一个table, 都会导致：
	
1. 栈顶新分配元素   
2. 绑定或赋值

还是为了重复一句话, 一个c value入栈就是进入了lua的世界, lua会生成一个对应的结构并管理起来, 从此就不再依赖这个c value。

lua value –> c value时, 是通过 lua_to* 族api实现, 很简单, 取出对应的c中的域的值就行了, 只能转化那些c中有对应值的lua value, 比如table就不能to c value, 所以api中夜没有提供 lua_totable这样的接口。

## Lua调用C++

有三种方法可以实现Lua调用c++中的代码。

第一种，直接写到lua源码中，然后重新编译lua文件。

在lua.c文件中加入自己的函数,别忘记在lua.h头文件中声明：

```c++
// This is my function  
static int getTwoVar(lua_State *L)  
{  
    // 向函数栈中压入2个值  
    lua_pushnumber(L, 10);  
    lua_pushstring(L,"hello");  
   
    return 2;  
}  
 
在pmain函数中，luaL_openlibs函数后加入以下代码：
//注册函数  
lua_pushcfunction(L, getTwoVar); //将函数放入栈中  
lua_setglobal(L, "getTwoVar");   //设置lua全局变量getTwoVar
//lua_register(L,"getTwoVar",getTwoVar);
```

lua_setglobal会弹出栈顶元素，并为其设置值。

编译之后可以直接调用函数了，但是不推荐这么做。

第二种：使用静态依赖的方式：

创建avg.lua:

```lua
avg, sum = average(10, 20, 30, 40, 50)  
print("The average is ", avg)  
print("The sum is ", sum)
```

创建cpp文件：

```c++
#include <stdio.h>  
extern "C" {  
#include "lua.h"  
#include "lualib.h"  
#include "lauxlib.h"  
}  
   
/* 指向Lua解释器的指针 */  
lua_State* L;  
static int average(lua_State *L)  
{  
    /* 得到参数个数 */  
    int n = lua_gettop(L);  
    double sum = 0;  
    int i;  
   
    /* 循环求参数之和 */  
    for (i = 1; i <= n; i++)  
    {  
        /* 求和 */  
        sum += lua_tonumber(L, i);  
    }  
    /* 压入平均值 */  
    lua_pushnumber(L, sum / n);  
    /* 压入和 */  
    lua_pushnumber(L, sum);  
    /* 返回返回值的个数 */  
    return 2;  
}  
   
int main ( int argc, char *argv[] )  
{  
    /* 初始化Lua */  
    L = lua_open();  
   
    /* 载入Lua基本库 */  
    luaL_openlibs(L);  
    /* 注册函数 */  
    lua_register(L, "average", average);  
    /* 运行脚本 */  
    luaL_dofile(L, "avg.lua");  
    /* 清除Lua */  
    lua_close(L);  
   
    /* 暂停 */  
    printf( "Press enter to exit…" );  
    getchar();  
    return 0;  
}
```
c++来带调用我们的lua文件，很别扭不是么？

第三种：使用动态库链接：

我们先新建一个dll工程，工程名为mLualib。（因此最后导出的dll也为mLualib.dll，然后编写我们的c++模块，以函数为例，我们先新建一个.h文件和.cpp文件。

h文件如下：（如果你不是很能明白头文件的内容，点击这里：http://blog.csdn.net/shun_fzll/article/details/39078971。）

```c
#pragma once  
extern "C" {  
#include "lua.h"  
#include "lualib.h"  
#include "lauxlib.h"  
}  
   
#ifdef LUA_EXPORTS  
#define LUA_API __declspec(dllexport)  
#else  
#define LUA_API __declspec(dllimport)  
#endif  
   
extern "C" LUA_API int luaopen_mLualib(lua_State *L);//定义导出函数
```

.cpp文件

```c
#include <stdio.h>  
#include "mLualib.h"  
static int averageFunc(lua_State *L)  
{  
    int n = lua_gettop(L);  
    double sum = 0;  
    int i;  
   
    /* 循环求参数之和 */  
    for (i = 1; i <= n; i++)  
        sum += lua_tonumber(L, i);  
   
    lua_pushnumber(L, sum / n);     //压入平均值  
    lua_pushnumber(L, sum);         //压入和  
   
    return 2;                       //返回两个结果  
}  
   
static int sayHelloFunc(lua_State* L)  
{  
    printf("hello world!");  
    return 0;  
}  
   
static const struct luaL_Reg myLib[] =   
{  
    {"average", averageFunc},  
    {"sayHello", sayHelloFunc},  
    {NULL, NULL}       //数组中最后一对必须是{NULL, NULL}，用来表示结束      
};  
   
int luaopen_mLualib(lua_State *L)  
{  
    luaL_register(L, "ss", myLib);  
    return 1;       // 把myLib表压入了栈中，所以就需要返回1  
}
```

```lua
require "mLualib"  
local ave,sum = ss.average(1,2,3,4,5)//参数对应堆栈中的数据  
print(ave,sum)  -- 3 15  
ss.sayHello()   -- hello world!
```

1. 我们编写了averageFunc求平均值和sayHelloFunc函数，

2. 然后把函数封装myLib数组里面，类型必须是luaL_Reg

3. 由luaopen_mLualib函数导出并在lua中注册这两个函数。

require库的时候：

```lua
local path = "mLualib.dll"    
local f = package.loadlib(path,"luaopen_mLualib")   -- 返回luaopen_mLualib函数  
f()                                                 -- 执行
```

所以当我们在编写一个这样的模块的时候，编写luaopen_xxx导出函数的时候，xxx最好是和项目名一样（因为项目名和dll一样）。

需要注意的是：函数参数里的lua_State是私有的，每一个函数都有自己的栈。当一个C/C++函数把返回值压入Lua栈以后，该栈会自动被清空。

# 总结
* Lua和C++是通过一个虚拟栈来交互的。
* C++调用Lua实际上是：由C++先把数据放入栈中，由Lua去栈中取数据，然后返回数据对应的值到栈顶，再由栈顶返回C++。
* Lua调C++也一样：先编写自己的C模块，然后注册函数到Lua解释器中，然后由Lua去调用这个模块的函数。上