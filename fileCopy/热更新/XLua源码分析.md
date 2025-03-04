# 明明白白的懂XLua

[toc]

## 热更新

热更新，就是在不重新去应用商店拉去更新包的情况下，从自己的服务器上拉取资源文件，然后进行差异对比读取，这样可以省很多的事情。

但是并不是你想更新什么就更新设么，首先那些基础的，支持你进行差异对比，拉取文件，需要编译的代码，这些东西都是不能做热更新的，我们称这一部分为基本部分，意思是支持你进行热更新和不能做热更新的部分，还有另一个部分就是业务部分，我们需要时常变动的部分。

用游戏来举例子：用户从网上下载下来的那个安装包叫做**母包**，它是我们游戏的最初版本，一般情况下这个包非常的完整，包含这个版本的基础部分和业务部分，当然，你也可以选择只放基础部分，等玩家进入游戏的时候，在把那些业务部分让玩家download下来。这个母包我们一般称之为大版本，是要放到应用商店中让玩家进行下载安装的。

接下来就是可热更新部分了，这部分可以放到我们的资源服务器上，然后每次玩家在登录的时候检测版本号是否和我们服务器上的一致，如果不一致，就告诉客户端你需要下载新的需要更新的资源。这样就可以一致保证玩家玩到的一致都是最新的版本。一本除了大版本，每次的热更新都是一个小的版本，大版本加上每个小版本就是我们完整的最新的游戏包了。

这些痕迹我们在生活中都可以看到，每次我们下载游戏的时候，发现进去一般都是需要更新的，这说明母包之后存在很多需要更新的小版本，然后我们还可以发现这些小版本都是一个接着一个的，这也说明，小版本也是按照顺序累积起来的。

## XLua的简单实用

在Lua中调用C#代码，在Lua中可以直接调用C#中的代码

```lua
CS.UnityEngine.Debug.Log('Hello world!')
```

这个就是通过表，和栈来实现通讯的。

**XLua中的CS变量的功能**

我们在Lua中，经常会使用一些不是Lua层设置和赋值的全局变量，这些变量，要么是自己通过GLobal获取之后进行调用的，要么是在LuaEnv中的init_lua string进行赋值的。像我们常用的CS，cast， typeof，xlua相关的函数，这里还有一点，就是xlua的有些函数是通过另一种途径设置的，在ObjectTranslator中，OpenLib添加方法。**而xlua这个全局变量是在xlua.c里面设置的**。下面的代码是在C#中的Lua代码，string类型的字符串，然后执行。



在下面的代码中有几个疑问？

1. .fqn

```lua
local metatable = {}
local rawget = rawget
local setmetatable = setmetatable
--关于这个方法的设定可以看后面，有解释
--这个方法是通过className
local import_type = xlua.import_type
local import_generic_type = xlua.import_generic_type
local load_assembly = xlua.load_assembly

--给元表设置元方法
function metatable:__index(key) 
    --为什么是 .fqn
    local fqn = rawget(self,'.fqn') --self指的是metatable
    fqn = ((fqn and fqn .. '.') or '') .. key
	-- 这个函数在ObjectTranslator中绑定，实现在StaticLuaCallBacks中
    local obj = import_type(fqn)

    if obj == nil then
        -- It might be an assembly, so we load it too.
        obj = { ['.fqn'] = fqn }
        setmetatable(obj, metatable)
    elseif obj == true then
        return rawget(self, key)
    end

    -- Cache this lookup
    rawset(self, key, obj)
    return obj
end

function metatable:__newindex()
    error('No such type: ' .. rawget(self,'.fqn'), 2)
end

-- A non-type has been called; e.g. foo = System.Foo()
function metatable:__call(...)
    local n = select('#', ...)
    local fqn = rawget(self,'.fqn')
    if n > 0 then
        local gt = import_generic_type(fqn, ...)
        if gt then
            return rawget(CS, gt)
        end
    end
    error('No such type: ' .. fqn, 2)
end

CS = CS or {}
setmetatable(CS, metatable)

typeof = function(t) return t.UnderlyingSystemType end
cast = xlua.cast
if not setfenv or not getfenv then
    local function getfunction(level)
        local info = debug.getinfo(level + 1, 'f')
        return info and info.func
    end

    function setfenv(fn, env)
        if type(fn) == 'number' then fn = getfunction(fn + 1) end
        local i = 1
        while true do
        local name = debug.getupvalue(fn, i)
        if name == '_ENV' then
            debug.upvaluejoin(fn, i, (function()
            return env
            end), 1)
            break
        elseif not name then
            break
        end

        i = i + 1
        end

        return fn
    end

    function getfenv(fn)
        if type(fn) == 'number' then fn = getfunction(fn + 1) end
        local i = 1
        while true do
        local name, val = debug.getupvalue(fn, i)
        if name == '_ENV' then
            return val
        elseif not name then
            break
        end
        i = i + 1
        end
    end
end

xlua.hotfix = function(cs, field, func)
    if func == nil then func = false end
    local tbl = (type(field) == 'table') and field or {[field] = func}
    for k, v in pairs(tbl) do
        local cflag = ''
        if k == '.ctor' then
            cflag = '_c'
            k = 'ctor'
        end
        local f = type(v) == 'function' and v or nil
        xlua.access(cs, cflag .. '__Hotfix0_'..k, f) -- at least one
        pcall(function()
            for i = 1, 99 do
                xlua.access(cs, cflag .. '__Hotfix'..i..'_'..k, f)
            end
        end)
    end
    xlua.private_accessible(cs)
end
xlua.getmetatable = function(cs)
    return xlua.metatable_operation(cs)
end
xlua.setmetatable = function(cs, mt)
    return xlua.metatable_operation(cs, mt)
end
xlua.setclass = function(parent, name, impl)
    impl.UnderlyingSystemType = parent[name].UnderlyingSystemType
    rawset(parent, name, impl)
end

local base_mt = {
    __index = function(t, k)
        local csobj = t['__csobj']
        local func = csobj['<>xLuaBaseProxy_'..k]
        return function(_, ...)
                return func(csobj, ...)
        end
    end
}
base = function(csobj)
    return setmetatable({__csobj = csobj}, base_mt)
end
```

## Lua中获取C#类对应的Table表

> XLua中有两种方式来实现Lua调用CS中的方法，一种是反射来调用，一种是生成适配的代码。

在获取对应的Lua表时候，使用的是import_type方法，也是在创建LuaEnv实例的时候进行注册的代码：

```c#
ObjectTranslator.cs
public void OpenLib(RealStatePtr L) {
    //获取到lua中的全局变量xlua，并把它push栈中，然后将import_type，这个函数名push栈中，之后把luacfunction push到栈中，最后rawset这个函数给xlua这个表
    if (0 != LuaAPI.xlua_getglobal(L, "xlua")){  throw new Exception("call xlua_getglobal fail!" + LuaAPI.lua_tostring(L, -1));} 
    LuaAPI.xlua_pushasciistring(L, "import_type");
    LuaAPI.lua_pushstdcallcfunction(L,importTypeFunction);
    LuaAPI.lua_rawset(L, -3); 
}
```

上面代码中的*importTypeFunction*是一个C#委托当Lua中是调用import_type时候Lua会调用对应的C方法(Lua调用CFunction的原理,请查找Lua手册)，最后会调用到对应的C#委托上来。

```c
XLua.c
LUA_API void luaopen_xlua(lua_State *L) {
   luaL_openlibs(L);
#if LUA_VERSION_NUM == 503
   luaL_newlib(L, xlualib);
   lua_setglobal(L, "xlua");
#else
   luaL_register(L, "xlua", xlualib);
    lua_pop(L, 1);
#endif
}
```

代码很简单，*luaopen_xlua*是一个c函数，属于xlua.dll在创建LuaEnv时候会调用。调用后会设置一个全局变量xlua，也就是ObjectTranslator类中获取的xlua变量。然后将键值对"import_type"=C#委托（**在c中是指针函数，在c#中是委托**），压入xlua表中。这样就能在inti_xlua.lua中调用import_type方法了。

## C#中找指定Type对应Lua表的实现

在C#的InportType方法中会尝试在缓存中获取对应的Type。如果Type为空，那说明是第一次尝试引用对应的Type，代码就会判断是使用适配代码还是反射模式，来生辰对应的表。

```c#
StaticLuaCallBacks.cs
ObjectTranslator.cs
//importTypeFunction = new LuaCSFunction(StaticLuaCallbacks.ImportType);
public static int ImportType(RealStatePtr L)
{
    try
    {
        ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        //需要查询的类名,在栈顶
        string className = LuaAPI.lua_tostring(L, 1);
        //查找C#对应的Type(此处还没去查找对应Lua的表)
        Type type = translator.FindType(className);
        if (type != null)
        {
            //这句查找Lua中Type对应的表
            if (translator.GetTypeId(L, type) >= 0)
            {
                LuaAPI.lua_pushboolean(L, true);
            }
            else
            {
                return LuaAPI.luaL_error(L, "can not load type " + type);
            }
        }
        else
        {
            LuaAPI.lua_pushnil(L);
        }
        return 1;
    }
}

internal int getTypeId(RealStatePtr L, Type type, out bool is_first, LOGLEVEL log_level = LOGLEVEL.WARN)
{
    int type_id;
    is_first = false;
    //查询是否缓存中有Type对应的Lua表,有就直接返回
    if (!typeIdMap.TryGetValue(type, out type_id)) // no reference
    {
        ...
        is_first = true;
        Type alias_type = null;
        aliasCfg.TryGetValue(type, out alias_type);
        LuaAPI.luaL_getmetatable(L, alias_type == null ? type.FullName : alias_type.FullName);
        if (LuaAPI.lua_isnil(L, -1)) //no meta yet, try to use reflection meta
        {
            LuaAPI.lua_pop(L, 1);
            //此处会去检查是使用反射还是生成适配代码的逻辑
            if (TryDelayWrapLoader(L, alias_type == null ? type : alias_type))
            {
                LuaAPI.luaL_getmetatable(L, alias_type == null ? type.FullName : alias_type.FullName);
            }
            else
            {
                throw new Exception("Fatal: can not load metatable of type:" + type);
            }
        }
        //循环依赖，自身依赖自己的class，比如有个自身类型的静态readonly对象。
        if (typeIdMap.TryGetValue(type, out type_id))
        {
            LuaAPI.lua_pop(L, 1);
        }
        else
        {
            ...
            LuaAPI.lua_pushvalue(L, -1);
            type_id = LuaAPI.luaL_ref(L, LuaIndexes.LUA_REGISTRYINDEX);
            LuaAPI.lua_pushnumber(L, type_id);
            LuaAPI.xlua_rawseti(L, -2, 1);
            LuaAPI.lua_pop(L, 1);
            ...
            //缓存type与其对应到lua中的表
            typeIdMap.Add(type, type_id);
        }
    }
    return type_id;
}
public bool TryDelayWrapLoader(RealStatePtr L, Type type)
{
            if (loaded_types.ContainsKey(type)) return true;
            loaded_types.Add(type, true);
            LuaAPI.luaL_newmetatable(L, type.FullName); //先建一个metatable，因为加载过程可能会需要用到
            LuaAPI.lua_pop(L, 1);
            Action<RealStatePtr> loader;
            int top = LuaAPI.lua_gettop(L);
            //此处如果已经缓存,那么就是生成适配代码注册,
            //这边的逻辑也是为了用的时候才实例化对应的.
            //这个delayWrap是个字典,他的键值对在XLua_Gen_Initer_Register__类实例化时候自动填充
            if (delayWrap.TryGetValue(type, out loader))
            {
                delayWrap.Remove(type);
                //将类方法,字段,成员等加载上来
                loader(L);
            }
            //那么这里就是反射的逻辑了
            else
            {
                 ...
                //用反射将类方法,字段,成员等加载上来
                Utils.ReflectionWrap(L, type, privateAccessibleFlags.Contains(type));
                ...
            }
            ...
            ...
            return true;
}
```

在生成完Type对应的Lua表后还需要设置到Lua上去

下面的代码简单来说就是用前面代码生成的table表设置到CS.UnityEngine[Debug]中：

`SetCSTable`这个函数会在Gen的Wrap代码的开始注册类的方法中调用或者`ReflectionWrap`中调用。

```c#
//loader(L)和Utils.ReflectionWrap(L, type, privateAccessibleFlags.Contains(type));中都会调用此函数来设置CS.UnityEngine[Debug]
public static void SetCSTable(RealStatePtr L, Type type, int cls_table)
{
   int oldTop = LuaAPI.lua_gettop(L);
   cls_table = abs_idx(oldTop, cls_table);
   LuaAPI.xlua_pushasciistring(L, LuaEnv.CSHARP_NAMESPACE);
   LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
   List<string> path = getPathOfType(type);
   for (int i = 0; i < path.Count - 1; ++i)
   {
      LuaAPI.xlua_pushasciistring(L, path[i]);
      if (0 != LuaAPI.xlua_pgettable(L, -2))
      {
         LuaAPI.lua_settop(L, oldTop);
         throw new Exception("SetCSTable for [" + type + "] error: " + LuaAPI.lua_tostring(L, -1));
      }
      if (LuaAPI.lua_isnil(L, -1))
      {
         LuaAPI.lua_pop(L, 1);
         LuaAPI.lua_createtable(L, 0, 0);
         LuaAPI.xlua_pushasciistring(L, path[i]);
         LuaAPI.lua_pushvalue(L, -2);
         LuaAPI.lua_rawset(L, -4);
      }
      else if (!LuaAPI.lua_istable(L, -1))
      {
         LuaAPI.lua_settop(L, oldTop);
         throw new Exception("SetCSTable for [" + type + "] error: ancestors is not a table!");
      }
      LuaAPI.lua_remove(L, -2);
   }
   LuaAPI.xlua_pushasciistring(L, path[path.Count - 1]);
   LuaAPI.lua_pushvalue(L, cls_table);
   LuaAPI.lua_rawset(L, -3);
   LuaAPI.lua_pop(L, 1);
   LuaAPI.xlua_pushasciistring(L, LuaEnv.CSHARP_NAMESPACE);
   LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
   ObjectTranslatorPool.Instance.Find(L).PushAny(L, type);
   LuaAPI.lua_pushvalue(L, cls_table);
   LuaAPI.lua_rawset(L, -3);
   LuaAPI.lua_pop(L, 1);
}
```

### 调用指定c#方法

***使用生成适配代码调用***，所有的生成的代码都在gen目录下：

```c#
UnityEngineDebugWrap.cs
public class UnityEngineDebugWrap
{
     public static void __Register(RealStatePtr L)
     {
          ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
          System.Type type = typeof(UnityEngine.Debug);
          //注册成员方法等
          Utils.BeginObjectRegister(type, L, translator, 0, 0, 0, 0);
          Utils.EndObjectRegister(type, L, translator, null, null,null, null, null);
          //注册类方法等即Static
          Utils.BeginClassRegister(type, L, __CreateInstance, 17, 3, 1);
          ...
          //注册一个名为Log的回调
          Utils.CLS_IDX(L, Utils.CLS_IDX, "Log", _m_Log_xlua_st_);
          ...
          Utils.EndClassRegister(type, L, translator);
     }



    [MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
    static int _m_Log_xlua_st_(RealStatePtr L)
    {
        //根据Log方法的参数数量来生成各种调用
       try {
          ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
          int gen_param_count = LuaAPI.lua_gettop(L);
          if(gen_param_count == 1&& translator.Assignable<object>(L, 1))
          {
              object _message = translator.GetObject(L, 1, typeof(object));
              UnityEngine.Debug.Log( _message );
              return 0;
          }
          if(gen_param_count == 2&& translator.Assignable<object>(L, 1)&& translator.Assignable<UnityEngine.Object>(L, 2))
          {
              object _message = translator.GetObject(L, 1, typeof(object));
              UnityEngine.Object _context = (UnityEngine.Object)translator.GetObject(L, 2, typeof(UnityEngine.Object));
              UnityEngine.Debug.Log( _message, _context );
              return 0;
          }
 
      } catch(System.Exception gen_e) {
          return LuaAPI.luaL_error(L, "c# exception:" + gen_e);
      }
      return LuaAPI.luaL_error(L, "invalid arguments to UnityEngine.Debug.Log!");
   }
}
注册代码如下
Utils.cs
public static void RegisterFunc(RealStatePtr L, int idx, string name, LuaCSFunction func)
{
  //这里的idx指的是就是CLS_IDX,就是cls_table,也就是SetCSTable设置的表
   idx = abs_idx(LuaAPI.lua_gettop(L), idx);
   //压入方法名
   LuaAPI.xlua_pushasciistring(L, name);
   //压入C#委托指针
   LuaAPI.lua_pushstdcallcfunction(L, func);
   LuaAPI.lua_rawset(L, idx);
}
```

***使用反射式调用***：

```c#
static int FixCSFunction(RealStatePtr L)
{
    try
    {
        ObjectTranslator translator = ObjectTranslatorPool.Instance.Find(L);
        //这边获取闭包中的upvalue值
        int idx = LuaAPI.xlua_tointeger(L, LuaAPI.xlua_upvalueindex(1));
        //GetFixCSFunction很简单就是return fix_cs_functions[index]; 
        //fix_cs_functions这个是在PushFixCSFunction时候添加的,PushFixCSFunction是在之前ReflectionWrap中调用的
        LuaCSFunction func = (LuaCSFunction)translator.GetFixCSFunction(idx);
        return func(L);
    }
    catch (Exception e)
    {
        return LuaAPI.luaL_error(L, "c# exception in FixCSFunction:" + e);
    }
}
```

当DoString到CS.UnityEngine.Debug.Log("hello world")时候，先从CS.UnityEngine.Debug[Log]获取到对应的value，在lua中这个值是一个function，那么就执行call，压入参数然后就开始调用了。如果是生成是适配代码的方式的话其对应的C#委托就是 _m_Log_xlua_st_(RealStatePtr L)了。但是如果是反射式调用的话,其对应的C#委托永远都是StaticLuaCallbacks.FixCSFunctionWraper这个委托，就是上面代码的FixCSFunction。当调用FixCSFunction后会从中取出upvalue，这个值是一个数字，是个索引。索引的是之前生成wrap时候缓存的方法。然后直接进行调用。



## 对象方法的调用

- 类的静态字段方法实现

在XLua中，对类的静态字段属性方法调用都是归属于cla_table的映射操作。

在生成适配代码的模式中国，XLua将一个类的静态方法名都作为一个Key存储在cls_table中，对应的值是一个委托，这个委托将会调用到C#中对应字段或者方法上，这个委托都是会在XLua的GenWrap代码阶段生成。

在反射模式中XLua也是讲一个雷的静态方法名作为一个key存储在cls_table中，但是其所有的值都对应的是同一个委托，这个委托是一个闭包函数，闭包函数中会有一个值，它索引的是背刺应该调用C#中的某个方法。

XLua对一个类的字段和属性实现与方法不同，XLua将字段名和属性名为Key存储在一个cls_getter的table中，而对用的值是一个委托，这个委托会调用到C#对应的属性和字段，但是要注意的是这个cls_getter table不会储存在cls_table中，而是作为闭包函数的一个参数保存起来，而且这个闭包函数会被作为cls_table的_index元方法。

```c#
public static void EndClassRegister(Type type, RealStatePtr L, ObjectTranslator translator)
{
    //获取栈顶Id
    int top = LuaAPI.lua_gettop(L);
    int cls_idx = abs_idx(top, CLS_IDX);//cls_table
    int cls_getter_idx = abs_idx(top, CLS_GETTER_IDX);//cls_get
    int cls_setter_idx = abs_idx(top, CLS_SETTER_IDX);//cls_set
    int cls_meta_idx = abs_idx(top, CLS_META_IDX);//cls_meta

    //begin cls index
    LuaAPI.xlua_pushasciistring(L, "__index");//压入__index 字符串，为设置__index元方法做准备
    LuaAPI.lua_pushvalue(L, cls_getter_idx);//压入cls_get Table
    LuaAPI.lua_pushvalue(L, cls_idx);//压入cls_table
    translator.Push(L, type.BaseType());//压入BaseType的地址
    LuaAPI.xlua_pushasciistring(L, LuaClassIndexsFieldName);//压入全局变量Key
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);//获取对应的全局变量值，并且弹出压入全局变量Key
    //生成cls_index方法,代码在下面一段分析，
    //只要知道这边会生成一个c方法，压入栈中
    //并且弹出之前压入的4个值(其实是弹出五个，还有一个是在gen_cls_index中压入的)。
    LuaAPI.gen_cls_indexer(L);
    //到这边其值栈中只有 top|__index|gen_cls_index，然后继续压入一个全局变量
    LuaAPI.xlua_pushasciistring(L, LuaClassIndexsFieldName);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);//store in lua indexs function tables
    //压入当前类型
    translator.Push(L, type);
    //压入-3的值，-3位置上现在是gen_cls_index
    LuaAPI.lua_pushvalue(L, -3);
    //设置键值对，其实就是 全局变量[type] = gen_cls_index，并且弹出type和栈顶gen_cls_index
    LuaAPI.lua_rawset(L, -3);
    //弹出全局变量
    LuaAPI.lua_pop(L, 1);
    //为cls_meta设置键值对 cls_meta[__index] = gen_cls_index,然后弹出键值对，
    //现在栈变为 top|
    LuaAPI.lua_rawset(L, cls_meta_idx);
    //end cls index
}
```

在Lua中，我们调用Object,name的时候，就会调用到Object对应的cls_table,然后name字段是不存在的，接着调用`__index`方法，上文中的`__index`方法对用的其实就是cls_indexer,实现如下：

```c#
//upvalue --- [1]:getters, [2]:feilds, [3]:base, [4]:indexfuncs, [5]:baseindex
//param   --- [1]: obj, [2]: key
LUA_API int cls_indexer(lua_State *L) {	
        //检查cls_get是否有值，没有值就代表当前类型没有get方法，然后就会尝试在父类查找
	if (!lua_isnil(L, lua_upvalueindex(1))) {
                //注意这边压入的不是upvalue2而是当前栈的2，是key，也就是name
		lua_pushvalue(L, 2);
                //在cls_get Table中查找name的值，
		lua_gettable(L, lua_upvalueindex(1));
		if (!lua_isnil(L, -1)) {//has getter
                        //我们已经知道值都是委托方法，所以这边直接调用获取值
			lua_call(L, 0, 1);
			return 1;
		}
	}
	//如果是当前类这边的upvalue2是cls_table所以肯定找不到字段和属性的，
        //但是如果已经是父类的话是会进行查找对应的静态方法的
	if (!lua_isnil(L, lua_upvalueindex(2))) {
		lua_pushvalue(L, 2);
		lua_rawget(L, lua_upvalueindex(2));
		if (!lua_isnil(L, -1)) {//has feild
			return 1;
		}
		lua_pop(L, 1);
	}
	//这个方法是一个Lazy的模式，如果需要在父类查找对应的字段和属性，
        //第一次会对upvalue5的设置好父类的index,然后自己置空，减少查询次数，直接使用upvalue5来进行查询
	if (!lua_isnil(L, lua_upvalueindex(3))) {
		lua_pushvalue(L, lua_upvalueindex(3));
		while(!lua_isnil(L, -1)) {
			lua_pushvalue(L, -1);
			lua_gettable(L, lua_upvalueindex(4));
			if (!lua_isnil(L, -1)) // found
			{
				lua_replace(L, lua_upvalueindex(5)); //baseindex = indexfuncs[base]
				lua_pop(L, 1);
				break;
			}
			lua_pop(L, 1);
			lua_getfield(L, -1, "BaseType");
			lua_remove(L, -2);
		}
		lua_pushnil(L);
		lua_replace(L, lua_upvalueindex(3));//base = nil
	}
	//查询父类的属性和字段upvalue5如果有值就是一个方法，所以参数就是obj,key然后查询。
	if (!lua_isnil(L, lua_upvalueindex(5))) {
		lua_settop(L, 2);
		lua_pushvalue(L, lua_upvalueindex(5));
		lua_insert(L, 1);
		lua_call(L, 2, 1);
		return 1;
	} else {
		lua_pushnil(L);
		return 1;
	}
}
```

**在反射模式中实现也是一样的，在此就不再赘述，字段属性的set也是与get一致，无非_\*index元方法变成了\*__newindex方法而已，也不再赘述。**

**对象字段方法实现**

同类型的对象其对字段属性方法的查找都是使用的同一个元表，元表存储在全局变量中，并且在每次创建对象的时候，根据对象类型，查找对应的元表，然后将对应的lua对象的元表设置为查找到的元表即可，完成操作映射。

```c#
public static void EndObjectRegister(Type type, RealStatePtr L, ObjectTranslator translator, LuaCSFunction csIndexer,
			LuaCSFunction csNewIndexer, Type base_type, LuaCSFunction arrayIndexer, LuaCSFunction arrayNewIndexer)
{
    //栈顶
    int top = LuaAPI.lua_gettop(L);
    int meta_idx = abs_idx(top, OBJ_META_IDX);//obj_meta
    int method_idx = abs_idx(top, METHOD_IDX);//obj_method
    int getter_idx = abs_idx(top, GETTER_IDX);//obj_get
    int setter_idx = abs_idx(top, SETTER_IDX);//obj_set
    //begin index gen
    //压入__index，准备__index元方法
    LuaAPI.xlua_pushasciistring(L, "__index");
    //压入值
    LuaAPI.lua_pushvalue(L, method_idx);
    //压入值
    LuaAPI.lua_pushvalue(L, getter_idx);
    //C#中的索引器的实现（比如List类型的索引器），暂时先忽略后续有时间，在来讨论实现
    if (csIndexer == null){LuaAPI.lua_pushnil(L);}
    else{LuaAPI.lua_pushstdcallcfunction(L, csIndexer);}
    //压入类型
    translator.Push(L, type == null ? base_type : type.BaseType());
    //压入全局变量
    LuaAPI.xlua_pushasciistring(L, LuaIndexsFieldName);
    LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);
    //对象数组的索引，注意跟上面的csIndex区别， 这边一样忽略，下次有时间讲
    if (arrayIndexer == null){LuaAPI.lua_pushnil(L);}
    else{LuaAPI.lua_pushstdcallcfunction(L, arrayIndexer);}
    //跟类中一样，也是生成闭包方法，弹出上面的6个值，还有一个是nil在C中压入的跟类中一样
    LuaAPI.gen_obj_indexer(L);
    if (type != null){
        //将键值对 type=gen_obj_index缓存到全局变量表中
        LuaAPI.xlua_pushasciistring(L, LuaIndexsFieldName);
        LuaAPI.lua_rawget(L, LuaIndexes.LUA_REGISTRYINDEX);//store in lua indexs function tables
        translator.Push(L, type);
        LuaAPI.lua_pushvalue(L, -3);
        LuaAPI.lua_rawset(L, -3);
        LuaAPI.lua_pop(L, 1);
    }
    //obj_meta[__index] = gen_obj_index
    LuaAPI.lua_rawset(L, meta_idx);
    //end index gen
}
```

gen_obj_indexer也是跟之前的一样，这个方法实现的是对子类和父类搜索，也是压入一个nil的值，给后续查找父类的索引器的占一个位置。

这边主要来看下obj*indexer方法，也就是gen_obj_indexr中生成的闭包方法，代码如下*

```c#
//upvalue --- [1]: methods, [2]:getters, [3]:csindexer, [4]:base, [5]:indexfuncs, [6]:arrayindexer, [7]:baseindex
//param   --- [1]: obj, [2]: key
LUA_API int obj_indexer(lua_State *L) {	
        //直接根据key查找对应的方法体然后返回
	if (!lua_isnil(L, lua_upvalueindex(1))) {
		lua_pushvalue(L, 2);
		lua_gettable(L, lua_upvalueindex(1));
		if (!lua_isnil(L, -1)) {//has method
			return 1;
		}
		lua_pop(L, 1);
	}
	//这边与上文的类获取get类似
	if (!lua_isnil(L, lua_upvalueindex(2))) {
		lua_pushvalue(L, 2);
		lua_gettable(L, lua_upvalueindex(2));
		if (!lua_isnil(L, -1)) {//has getter
			lua_pushvalue(L, 1);
			lua_call(L, 1, 1);
			return 1;
		}
		lua_pop(L, 1);
	}
	
	//其实意思就是如果key是数字那么就根据下标去调用array函数获取对应值，看点主要在arrayIndex方法中
	if (!lua_isnil(L, lua_upvalueindex(6)) && lua_type(L, 2) == LUA_TNUMBER) {
		lua_pushvalue(L, lua_upvalueindex(6));
		lua_pushvalue(L, 1);
		lua_pushvalue(L, 2);
		lua_call(L, 2, 1);
		return 1;
	}
	//与arrayIndex类似
	if (!lua_isnil(L, lua_upvalueindex(3))) {
		lua_pushvalue(L, lua_upvalueindex(3));
		lua_pushvalue(L, 1);
		lua_pushvalue(L, 2);
		lua_call(L, 2, 2);
		if (lua_toboolean(L, -2)) {
			return 1;
		}
		lua_pop(L, 2);
	}
	//父类indexfuncs查找不赘述
	if (!lua_isnil(L, lua_upvalueindex(4))) {
		lua_pushvalue(L, lua_upvalueindex(4));
		while(!lua_isnil(L, -1)) {
			lua_pushvalue(L, -1);
			lua_gettable(L, lua_upvalueindex(5));
			if (!lua_isnil(L, -1)) // found
			{
				lua_replace(L, lua_upvalueindex(7)); //baseindex = indexfuncs[base]
				lua_pop(L, 1);
				break;
			}
			lua_pop(L, 1);
			lua_getfield(L, -1, "BaseType");
			lua_remove(L, -2);
		}
		lua_pushnil(L);
		lua_replace(L, lua_upvalueindex(4));//base = nil
	}
	//递归查找父类
	if (!lua_isnil(L, lua_upvalueindex(7))) {
		lua_settop(L, 2);
		lua_pushvalue(L, lua_upvalueindex(7));
		lua_insert(L, 1);
		lua_call(L, 2, 1);
		return 1;
	} else {
		return 0;
	}
}
```

