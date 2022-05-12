# 模块/包

## 基础知识

在本文开始之前  
可先浏览 [Lua 模块与包-菜鸟教程](http://www.runoob.com/lua/lua-modules-packages.html)  
粗略了解下关于 Lua 模块/包 的基础知识  
特别是 package.path 的相关内容

## 我的研究

在 [VS 调试 Lua](vsdebug.md) 中  
得知了可以向 VS 项目中添加 `命令参数`  
以达到执行命令行 `$ lua ~/Projects/Lua51/script/main.lua` 类似的效果

但在我研究的过程中  
总是需要运行测试不同的 lua 代码  
如果要因此而频繁去修改 `命令参数` 配置  
显然不合适

因此我把要测试的 lua 代码文件  
在 main.lua 中 require 进来  
这样可不修改 lua 源码工程的配置  
即可执行不同的 lua 文件

## require lua 文件

解压的 lua 源码目录下的 test 目录  
有很多测试用的 lua 脚本代码  
把 test 目录复制到 `Lua51/script` 目录下  
编辑 `script/main.lua` ，内容如下：

```lua
modules = {
    --"test.bisect",          --bisection method for solving non-linear equations
    --"test.cf",              --temperature conversion table (celsius to farenheit)
    --"test.echo",            --echo command line arguments
    --"test.env",             --environment variables as automatic global variables
    --"test.factorial",       --factorial without recursion
    --"test.fib",             --fibonacci function with cache
    --"test.fibfor",          --fibonacci numbers with coroutines and generators
    --"test.globals",         --report global variable usage
    --"test.hello",           --the first program in every languag
    --"test.life",            --Conway's Game of Life
    --"test.luac",            --bare-bones luac
    "test.printf",          --an implementation of printf
    --"test.readonly",        --make global variables readonly
    --"test.sieve",           --the sieve of of Eratosthenes programmed with coroutines
    --"test.sort",            --two implementations of a sort function
    --"test.table",           --make table, grouping all data for the same item
    --"test.trace-calls",     --trace calls
    --"test.trace-globals",   --trace assigments to global variables
    --"test.xd",              --hex dump
}

for i, v in ipairs(modules) do
    require(v);
end
```

注意，代码中除了 test.printf 一行，其他都注释了

此时运行会报如下错误：

```text
module 'test.printf' not found:
        no field package.preload['test.printf']
        no file '.\test\printf.lua'
        no file 'X:\Lua51\x64\Debug\lua\test\printf.lua'
        no file 'X:\Lua51\x64\Debug\lua\test\printf\init.lua'
        no file 'X:\Lua51\x64\Debug\test\printf.lua'
        no file 'X:\Lua51\x64\Debug\test\printf\init.lua'
        no file '.\test\printf.dll'
        no file 'X:\Lua51\x64\Debug\test\printf.dll'
        no file 'X:\Lua51\x64\Debug\loadall.dll'
        no file '.\test.dll'
        no file 'X:\Lua51\x64\Debug\test.dll'
        no file 'X:\Lua51\x64\Debug\loadall.dll'
stack traceback:
        [C]: in function 'require'
        X:\Lua51\script\main.lua:24: in main chunk
        [C]: ?
```

此时若在 main.lua 中添加如下代码运行：

```text
print(package.path);
```

输出如下：

```text
.\?.lua;X:\Lua51\x64\Debug\lua\?.lua;X:\Lua51\x64\Debug\lua\?\init.lua;X:\Lua51\x64\Debug\?.lua;X:\Lua51\x64\Debug\?\init.lua
```

显然没有 `Lua51/script` 目录

## package.path

从 [Lua 参考手册](http://www.runoob.com/manual/lua53doc/manual.html#pdf-package.pat) 中得知：

```text
package.path
在启动时，Lua 用环境变量 LUA_PATH_5_1 或环境变量 LUA_PATH 来初始化这个变量。
或采用 luaconf.h 中的默认路径。
```

luaconf.h 中有这段代码：

```c
/*
** In Windows, any exclamation mark ('!') in the path is replaced by the
** path of the directory of the executable file of the current process.
*/
#define LUA_LDIR    "!\\lua\\"
#define LUA_CDIR    "!\\"
#define LUA_PATH_DEFAULT  \
        ".\\?.lua;"  LUA_LDIR"?.lua;"  LUA_LDIR"?\\init.lua;" \
                     LUA_CDIR"?.lua;"  LUA_CDIR"?\\init.lua"
```

其中的注释翻译如下：

```text
在 Windows 中，路径中的所有感叹号（'!'）都会被替换为当前进程的可执行文件的目录路径。
```

`X:\Lua51\x64\Debug` 这个路径因此而来

那在路径中 . 所代表的当前路径是什么路径？  
在 main.lua 中加入以下代码：

```lua
local f = io.open("./lua_path.tmp", "w");
f:close();
```

在命令行中输入：

```text
$ cd ~/Desktop/
$ lua ~/Projects/Lua51/script/main.lua
```

会在 `~/Desktop/` 目录下生成 lua\_path.tmp 文件  
如果运行 VS 的 lua 项目  
则会在 `X:\Lua51\lua\` 目录下生成 lua\_path.tmp 文件  
推论得出：

```text
路径中 . 所代表的当前路径为执行 lua 程序时所在的目录路径
```

## 把 script 目录路径加入到 package.path

有两种方法

1. 在 lua 脚本中修改 package.path 字符串
2. 修改 lua 源码中对 package.path 变量的赋值

我采用的是第一种方法  
在 main.lua 文件开头添加如下代码即可：

```lua
local info = debug.getinfo(1, "S");
local path = info.short_src;
local dir = string.match(path, "^.*[/\\]");

if dir then
    package.path = package.path .. ";" .. dir .. "?.lua";
end
```

