---
layout:     post
title:      Lua 学习 chapter28
subtitle:   扩展应用
date:       2019-8-1
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 基础知识
2. 操作表
3. 一些简便方法
4. 调用lua函数


> 回顾复习加巩固，自己应该认真地学习复习和巩固，并不是只有学习新知识是自己最重要的，而是复习和巩固才是自己最重要的，所以自己每天必须抽出一定的时间对自己的学习进行复习和巩固。

## 基础知识

lua可以作为配置文件来供c语言使用，例如我们来定义一个窗口的大小。

```
width = 200
height = 300
```

然后我们使用LuaAPI来指挥lua语言解析该文件，并获取width和height的值。

* luaL_loadfile(lua_State *L, fname):加载文件并且编译，返回一个执行这个文件的函数，放到栈顶
* lua_pcall(L,0,0,0):调用栈中的函数，**第二个参数为函数参数的个数，第三个参数为返回值的个数，第四个参数为错误处理函数**。在压入结果之前，pcall函数会将参数和函数出栈，将返回值从第一个按照顺序压入栈中。至于最后一个参数为零，则lua会把错误信息作为string压倒栈中，如果是函数，第四个参数为它在栈中的索引，一般这个错误处理函数应该在被压入栈且位于待调用函数之下。
* lua_getglobal(lua_State \*L, const char \*key):根据key值获取全局变量，并放到栈顶。

```c
#include<stdio.h>
#include<string.h>
#include<stdarg.h>
#include<stdlib.h>
extern "C" {
#include <lua.h>
#include <lauxlib.h>
#include <lualib.h>
}

int getglobint(lua_State* L, const char* var);
void load(lua_State* L, const char* fname, int* w, int* h);
void error(lua_State* L, const char* fmt, ...);

int main(void)
{
	lua_State* L = luaL_newstate();//打开lua
	luaL_openlibs(L);//打开标准库

	int width, height;
	load(L, "config", &width, &height);
	printf("%d %d", width, height);
	return 0;
}

int getglobint(lua_State* L, const char* var)
{
	int isnum, result;
	lua_getglobal(L, var);//将相应的全局变量的值压入到栈中。
	result = (int)lua_tointegerx(L, -1, &isnum);
	if (!isnum)
	{
		error(L, "'%s' should be a number \n", var);
	}
	lua_pop(L, 1);
	return result;
}

void load(lua_State* L, const char* fname, int* w, int* h)
{
	if (luaL_loadfile(L,fname) || lua_pcall(L,0,0,0))//loadfile加载代码，pcall运行编译后的代码段
	{
		error(L, "cannot  run config. file %s", lua_tostring(L,-1));
	}
	*w = getglobint(L, "width");
	*h = getglobint(L, "height");
}

void error(lua_State* L, const char* fmt, ...)
{
	va_list argp;
	va_start(argp, fmt);
	vfprintf(stderr, fmt, argp);
	va_end(argp);
	lua_close(L);
	exit(EXIT_FAILURE);
}


}
```

## 操作表

现在我们要为每个窗口配置一种背景色，假设最终的颜色格式是由三个数字分量组成的RGB颜色。

```
width = 200
height = 300
BLUE = {red = 0,green = 0,blue = 1.0}
background = BLUE
```

我们可以对表进行操作，具体用到的函数：

* lua_pushstring(L,key)
* lua_gettable(L,-2) //-2是表的索引，它会pop出key，然后根据key取值，并把值放到栈中
* lua_getfield(L,-1,key)//相当于上面的两步，这个函数会返回结果的类型
* lua_pushnumber(L,val)
* lua_pushstring(L,key)
* lua_settable(L,-3)// -3是表的索引，它会pop出val，key，然后给表设置值。
* lua_setfield(L,-2,key)// 相当于上面的两步，它会pop出val然后根据key设置值。
* lua_createtable(L,int narr,int nrec):narr,表示连续的值个数，nrec表示hash的值个数。

```c
#include<stdio.h>
#include<string.h>
#include<stdarg.h>
#include<stdlib.h>
extern "C" {
#include <lua.h>
#include <lauxlib.h>
#include <lualib.h>
}

const int MAX_COLOR = 255;

struct ColorTable
{
	const char* name;
	unsigned char red, green, blue;
}colortable[] = {
	{"WHITE", MAX_COLOR, MAX_COLOR, MAX_COLOR},
	{"RED", MAX_COLOR, 0, 0},
	{"GREEN", 0, MAX_COLOR, 0},
	{"BLUE", 0, 0, MAX_COLOR},
	{NULL, 0, 0, 0}
};

int getglobint(lua_State* L, const char* var);
void load(lua_State* L, const char* fname, int* w, int* h);
void error(lua_State* L, const char* fmt, ...);
void getColor(lua_State* L, const char* what, int* r, int* g, int* b);
int getcolorfield(lua_State* L, const char* key);
void setcolor(lua_State* L,struct ColorTable *ct);
void setcolorfield(lua_State* L, const char* index, int value);


int main(void)
{
	lua_State* L = luaL_newstate();//打开lua
	luaL_openlibs(L);//打开标准库

	int width, height, red, green, blue;
	load(L, "config", &width, &height);
	printf("%d %d \n", width, height);
	getColor(L, "background", &red, &green, &blue);
	printf("%d\t%d\t%d\n", red, green, blue);
	int i = 0;
	while (colortable[i].name != NULL)
	{
		setcolor(L, &colortable[i++]);
	}
	return 0;
}

int getglobint(lua_State* L, const char* var)
{
	int isnum, result;
	lua_getglobal(L, var);
	result = (int)lua_tointegerx(L, -1, &isnum);
	if (!isnum)
	{
		error(L, "'%s' should be a number \n", var);
	}
	lua_pop(L, 1);
	return result;
}

void load(lua_State* L, const char* fname, int* w, int* h)
{
	if (luaL_loadfile(L,fname) || lua_pcall(L,0,0,0))
	{
		error(L, "cannot  run config. file %s", lua_tostring(L,-1));
	}
	*w = getglobint(L, "width");
	*h = getglobint(L, "height");
}

void error(lua_State* L, const char* fmt, ...)
{
	va_list argp;
	va_start(argp, fmt);
	vfprintf(stderr, fmt, argp);
	va_end(argp);
	lua_close(L);
	exit(EXIT_FAILURE);
}

void getColor(lua_State* L, const char* what, int* r, int* g, int* b)
{
	lua_getglobal(L, what);
	if (!lua_istable(L,-1))
	{
		error(L, "'%s' is not a table", what);
	}

	*r = getcolorfield(L, "red");
	*g = getcolorfield(L, "green");
	*b = getcolorfield(L, "blue");
}

int getcolorfield(lua_State* L, const char* key)
{
	int result, isnum;
	lua_getfield(L, -1, key);
	result = (int)(lua_tonumberx(L, -1, &isnum) * MAX_COLOR);
	if (!isnum)
	{
		error(L, "invalid component '%s' in color", key);
	}
	lua_pop(L, 1);
	return result;
}

void setcolor(lua_State* L, ColorTable* ct)
{
	lua_newtable(L);
	setcolorfield(L, "red", ct->red);
	setcolorfield(L, "green", ct->green);
	setcolorfield(L, "blue", ct->blue);
	lua_setglobal(L, ct->name);
}



void setcolorfield(lua_State* L, const char* index, int value)
{
	lua_pushstring(L, index);
	lua_pushnumber(L, double(value) / MAX_COLOR);
	lua_settable(L, -3);
	//lua_setfield(L, -2, index); lua_setfield是将pushstring和settable结合起来的
}

```

## 调用lua函数

```c
//一个通用的调用函数
//这里vl其实就是一个指针，指向了参数的地址。
//
//va_start()所做的就是让vl指向函数的最后一个确定的参数（声明程序中是sig）的下一个参数的地址。
//
//va_arg()所做的就是根据vl指向的地址，和第二个参数所确定的类型，将这个参数的中的数据提取出来，作为返回值，同时让ap指向下一个参数。
//
//va_end()所做的就是让vl这个指针指向0。
void call_va(lua_State* L, const char* func, const char* sig, ...) {
	va_list vl;
	int narg, nres; //参数和结果的个数

	va_start(vl, sig);
	lua_getglobal(L, func);

	//压入参数并计数
	for (narg = 0; *sig; narg++)
	{
		luaL_checkstack(L, 1, "too many argument");
		switch (*sig++)
		{
		case 'd':
			lua_pushnumber(L, va_arg(vl, double));
			break;
		case 'i':
			lua_pushinteger(L, va_arg(vl, int));
			break;
		case 's':
			lua_pushstring(L, va_arg(vl, char *));
			break;
		case '>':
			goto endargs;
		default:
			error(L,"invalid option(%c)", *(sig - 1));
		}
	}
	endargs:

	nres = strlen(sig);
	if (lua_pcall(L,narg,nres,0) != 0 )
	{
		error(L, "error calling '%s' : %s", func, lua_tostring(L, -1));
	}

	//获取结果
	nres = -nres;
	while (*sig)
	{
		switch (*sig++) {
			case 'd': {
				int isnum;
				double n = lua_tonumberx(L, nres, &isnum);
				if (!isnum)
				{
					error(L, "wrong result type");
				}
				*va_arg(vl, double*) = n;
				break;
			}
			case 'i': {
				int isnum;
				double n = lua_tointegerx(L, nres, &isnum);
				if (!isnum)
				{
					error(L, "wrong result type");
				}
				*va_arg(vl, int*) = n;
				break;
			}
			case 's': {
				const char *s = lua_tostring(L, nres);
				if (s == NULL)
				{
					error(L, "wrong result type");
				}
				*va_arg(vl, const char **) = s;
				break;
			}
			default:
				error(L, "invalid option (%c)", *(sig - 1));
		}
		nres++;
	}

	va_end(vl);
}
void loadfile(lua_State* L, const char* fname)
{
	if (luaL_loadfile(L, fname) || lua_pcall(L, 0, 0, 0))
	{
		error(L, "cannot  run config. file %s", lua_tostring(L, -1));
	}
}
//在main函数里面调用
loadfile(L, "test.lua");
int sum;
call_va(L, "f", "ii>i", 1, 2, &sum);
printf("sum is %d", sum);
```