---
layout:     post
title:      Lua 学习 chapter29
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
1. 前言
2. c函数
3. 延续
4. c模块


> 生活总需要一点仪式感，然后慢慢的像那个趋向完美的自己靠近。

## 前言

**Lua调用c函数时，我们必须注册该函数，即必须以一种恰当的方式为lua提供该c函数的地址**。

lua调用c函数时，也使用了一个与c语言调用Lua函数时相同类型的栈，**c函数从栈中获取参数，并将结果压入到栈中**。

需要注意的是，这个栈不是一个全局结构；**每个函数都有其私有的局部栈**。当Lua调用一个c函数时，第一个参数总是位于这个局部栈中，索引为1的位置。即使一个c函数调用了lua代码，而且lua代码用再次调用了同一个的c函数，这些调用每次只会看到本次调用自己的私有栈，其中索引为1的位置就是第一个参数。

流程是这样的：

1. Lua可以让程序员开发在Lua脚本中调用C函数的接口，这个接口称做LuaGlue函数，因为他们可以在Lua环境中整合C的功能。
2. Lua API提供了函数让C++代码也可以直接调用Lua函数，还提供了方法可以传递字符和长文字给Lua解释。

## C函数

随便写一个c的函数，该函数求一个数的正选值。

```c
static int l_sin(lua_State *L)
{
	double d = lua_tonumber(L,1);//获取参数
	lua_pushnumber(L,sin(d));//将结果入栈
	return 1;//返回值的个数
}
```

所有在lua中注册的函数都必须使用一个相同的原型，该原型就是定义在lua.h中的lua_CFunction，即所有的c函数如果要给lua进调用，只能用这样的函数来封装：

```c
typedef int(*lua_CFunction) (lua_State *L)

```

从C语言角度看，这个函数只有一个指向lua状态类型的指针作为参数，返回值为一个整型数，代表压入栈中的返回值的个数。因此，该函数在压入结果前无需清空栈。在该函数返回后，lua会自动报错返回值并清空整个栈。

**在lua中，调用这个函数前，还需要通过lua_pushfunction注册该函数。函数lua_pushfunction会获取一个指向c函数的指针，然后再lua中创建一个“function”类型，代表注册的函数。一旦完成注册，c函数就可以像其他lua函数一样行事了**。

一种快速测试l_sin的方法是，将其代码放到简单解释器中，并将线面的代码添加到luaL_openlibs调用的后面：

```c
lua_pushfunction(L,l_sin);
lua_setglobal(L,"mysin")
```

## 延续

通过lua_pcall和lua.call，一个被lua调用的c函数也可以回调lua函数。标准库中有一些函数就是这么做的：table.sort调用了排序函数，string.gsub调用了替换含糊，pcall和xpcall以保护模式来调用函数。如果你还记得lua代码本身就是被c代码(宿主）调用的，那么你应该知道调用顺序类似于：**C(宿主)调用Lua(脚本)，lua(脚本)又调用了C(库),C(库)又调用了lua(回调)**.

一般lua语言可以处理这种调用顺序，但是处理协程的话会有问题。

lua语言中，每个协程都有自己的栈，其中保存了改协程所挂起调用的信息。具体的说，就是该栈中存储了每一个调用的返回地址，参数以及局部变量。对于lua函数的调用，解释器只需要这个栈即可，我们将其称为软栈。然而对于c函数的调用，解释器必须使用c语言栈。毕竟c函数的返回地址和局部变量都位于c语言栈中。

对于解释器来说，拥有多个软栈并不难，但是ISO C的运行环境只能拥有一个内部栈。因此lua中的协程无法保存c函数的执行：如果一个c函数位于从resume到对应yield的调用路径中，那么lua无法保存c函数的状态以便下次resume时恢复状态。

```lua
co = corourine.wrap(function()
	print(pcall(corourine.yield))
	end)
co()

-->false attempt to yield across metamethod/C-call boundary
```

函数pcall是一个c语言函数；因此，lua5.1不能将其挂起，因为ISO C无法挂起一个C函数并在之后恢复期运行。

在lua5.2版本中，用延续来改善对这个问题的处理。lua5.2使用长跳转实现了yield，并使用相同的方式实现了错误处理。长跳转简单的抛弃可以指定一个延续函数foo_K，该函数也是一个c函数，在要恢复goo的执行时他就会被调用。也就是说，当解释器发现它应该恢复函数foo的执行时，如果长跳转已经丢弃了c语言栈中有关foo的信息，则调用foo_k来代替。

```c
static int luaB_pcall(lua_State *L)
{
	int status;
	lua_checkany(L,1);//至少一个参数
	status = lua_pcall(L,lua_gettop(L) - 1, LUA_MULTRET, 0 );//
	lua_pushboolean(L,(status==LUA_OK));//状态
	lua_insert(L,1); //状态是第一个结果
	return lua_gettop(L); //返回状态和所有结果
}

```

如果程序正在通过lua_pcall被调用的函数yield，那么后面就不可能恢复luaB_pcall的执行。因此，如果我们在保护模式的调用下试图yield时，解释器就会抛出异常。lua5.3使用的基本类似于下面的方式。

```c
static int finishpcall(lua_State *L, int status, intptr_t ctx)
{
	(void) ctx;
	status = (status != LUA_OK && status != LUA_YIELD);
	lua_pushboolean(L, (status == 0));
	lua_insert(L,1);
	return lua_gettop(L);
}

static int luaB_pcall(lua_State *L)
{
	int status;
	lua_checkany(L,1);//至少一个参数
	status = lua_pcall(L,lua_gettop(L) - 1, LUA_MULTRET, 0, 0,finishpcall);//
	return finishpcall(L,status,0);
}


```

## c模块

**lua模块就是一个代码段**，其中定义了一下lua函数并将其存储在恰当的地方(通常是表中的元素).**为lua编写的c语言模块可以模仿这种行为**。除了c函数的定义外，**c模块还必须定义一个特殊函数**，这个**特殊函数相当于lua库中的主代码段**，**用于注册模板中的所有c函数**，并将他们存储在恰当的地方(通常也是表中的元素)。与lua的主代码段一样，这个函数还应该初始化模块中所有需要初始化的其他东西。

lua通过注册过程感知到c函数，一旦一个c函数用lua表示和存储，lua就会通过对其地址(就是我们注册函数时提供的lua的信息)的直接引用来调用它。换句话说，一旦一个**c函数完成注册**，**lua调用它试就不在依赖于其函数名、包的位置以及可见性规则**。通常，一**个c模块中只有一个用于打开库的公共函数**；其他的所有函数都是私有的，在c语言中被声明为static。

当我们使用c函数扩展lua程序时，将代码设计为一个c模块是一个不错的想法。因为即使我们现在只想注册一个函数，但迟早总会需要其他的函数。通常，辅助库为这项工作提供了一个辅助函数。宏**luaL_newlib接收一个由c函数及其对应函数名组成的数组**，并将这下函数注册到一个新表中。eg:假设我们要用一个之前顶一个函数l_dir创建一个库。

```c
static int l_dir(lua_State *L)
{
	DIR *dir;
	struct dirent *entry;
	int i;
	const char *path = luaL_checkstring(L,1);
	dir = opendir(path);
	if (dir == NULL)
	{
		lua_pushnil(L);
		lua_pushstring(L,strerror(errno));
		return 2;
	}

	lua_newtable(L);
	i = 1;
	while((entry = readdir(dir) != NULL) {
		lua_pushinteger(L,i++);
		lua_pushstring(L,entry->name);
		lua_settable(L,-3);
	}
	closedir(dir);
	return 1;
}
```

然后，声明一个数组，这个数组包含了模块中所有的函数及其名称。数组元素的类型为luaL_Reg，该类型是由两个字段组成的结构体，这个两个字段分别是函数名和函数指针。

```c
static const struct luaL_Reg mylib[] = {
	{"dir", l_dir},
	{NULL,NULL} //哨兵
	
}
```
上面的例子只声明了一个函数，数组的最后一个元素永远是{NULL,NULL},并以此表示数组的结尾。最后我们声明一个函数luaL_newlib声明一个主函数：

```c
int luaopen_mylib(lua_State *L)
{
	luaL_newlib(L, mylib);
	return 1;
}
```

对函数lua_newLib的调用会创建一个表，并使用由数组mylib指定的"函数名-函数指针"填充这个新创建的表，当luaL_newlib返回时，它把这个新创建的表留在栈中，在表中它打开了这个库。然后，函数luaopne_mylib返回1，表示再将这个表返回给lua。

然后我们将其链接到解释器中，我们用代码创建一个动态链接库，并将这个库放到c语言路径中的某个地方，之后就可以require在乱终直接加载这个模块了。

```lua
local mylib = require "mylib"
```

上述的语句会将动态库mylib链接到lua，查找函数luaopen_mylib，将其注册为一个c语言函数，探后调用它一打开模块。

动态链接器必须知道函数luaopen_mylib的名字才能找到他。它总是寻找名为"luaopne_+模块名"这样的函数。因此如果我们的模块名为mylib，那么该函数应该命名为luaopen_mylib。

如果解释器不支持动态链接，需要同新库一起重新编译lua语言。除了重新编译，还需要以某种方式告诉独立解释器，他应该再打开一个新状态时打开这个库。一个简单的做法是把luaopen_mylib添加到由luaLopenlibs打开的标准库列表中，这个列表位于文件**linit.c**中。

