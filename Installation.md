# Lua 的源码编译与安装

[LuaReadMe]: http://www.lua.org/manual/5.3/readme.html
[LuaReadMeCn]: https://cloudwu.github.io/lua53doc/
[DownMinGW]: https://sourceforge.net/projects/mingw/files/latest/download?source=files
[AllLuaVersions]: http://www.lua.org/versions.html
[Lua515]: http://www.lua.org/ftp/lua-5.1.5.tar.gz
[InstallLua]: http://www.runoob.com/lua/lua-environment.html


## 下载 Lua 源码
[Lua 所有版本的源码][AllLuaVersions]  
本例使用的是 [Lua 5.1.5][Lua515]  
下载解压后的目录结构如下：
```
doc
etc
src
test
COPYRIGHT
HISTORY
INSTALL
Makefile
README
```


## Lua 安装
[Lua 各平台的快捷安装方式][InstallLua]


## 其他
### 阅读 Lua 官方 ReadMe
如果感兴趣或者安装步骤上出现问题，可阅读 [Lua 官网 ReadMe][LuaReadMe]，了解编译 Lua 的详细步骤  
[Lua 官方 ReadMe 中文版][LuaReadMeCn]

### Windows 上编译 Lua

我研究了两种编译方式 `利用 VS 编译` 和 `利用 MinGW 编译`  
由于之后利用 VS 断点调试研究 Lua 源码是我的主要手段  
故以下先介绍 `利用 VS 编译`

##### 利用 VS 编译


##### 利用 MinGW 编译
###### 安装 MinGW
下载并安装 [MinGW][DownMinGW]  
安装完成后勾选 `Baseic Setup` 中的除了 `mingw32-gcc-objc` 的所有选项  
点击 `Apply Changes` 按钮  
等待安装完成后即可关闭 `MinGW Installation Manager`  
  
在 `环境变量`->`系统变量` 中的 `Path` 变量中添加 `x:\MinGW\bin`  
即添加 `MinGW` 安装目录下的 `bin` 目录路径  

打开 Lua 源码目录下的 `src/Makefile` 文件  
找到并删除字符串 `# DLL needs all object files`  
  
回到 Lua 源码目录，`Shift`+`右键` 弹出菜单列表  
选择 `在此处打开命令窗口`  
在命令窗口中输入 `mingw32-make mingw`  
这里参照了 [Lua 官网 ReadMe][LuaReadMe] 中所提及的 `make xxx` 命令  
`xxx` 为平台名，Makefile 提供的平台名有：
```
aix
ansi
bsd
freebsd
generic
linux
macosx
mingw
posix
solaris
```
  
等待运行完成后，在 `src` 目录下已生成各种 .o 文件及解释器 lua.exe 和 编译器 luac.exe  
  
没有修改 `src/Makefile` 则会报以下错误：
```
...
gcc -shared -o lua51.dll lapi.o lcode.o ldebug.o ldo.o ldump.o lfunc.o lgc.o llex.o lmem.o lobject.o lopcodes.o lparser.o lstate.o lstring.o ltable.o
ldblib.o liolib.o lmathlib.o loslib.o ltablib.o lstrlib.o loadlib.o linit.o     # DLL needs all object files
gcc: error: #: No such file or directory
gcc: error: DLL: No such file or directory
gcc: error: needs: No such file or directory
gcc: error: all: No such file or directory
gcc: error: object: No such file or directory
gcc: error: files: No such file or directory
Makefile:51: recipe for target 'lua51.dll' failed
mingw32-make[2]: *** [lua51.dll] Error 1
mingw32-make[2]: Leaving directory 'X:/lua-5.1.5/src'
Makefile:107: recipe for target 'mingw' failed
mingw32-make[1]: *** [mingw] Error 2
mingw32-make[1]: Leaving directory 'X:/lua-5.1.5/src'
Makefile:56: recipe for target 'mingw' failed
mingw32-make: *** [mingw] Error 2
```

接着参照类 Unix 平台上的安装到系统默认目录的命令 `make xxx install`  
在命令窗口输入 `mingw32-make mingw install`  
结果报错：
```
cd src && mingw32-make mingw
mingw32-make[1]: Entering directory 'X:/lua-5.1.5/src'
mingw32-make "LUA_A=lua51.dll" "LUA_T=lua.exe" \
"AR=gcc -shared -o" "RANLIB=strip --strip-unneeded" \
"MYCFLAGS=-DLUA_BUILD_AS_DLL" "MYLIBS=" "MYLDFLAGS=-s" lua.exe
mingw32-make[2]: Entering directory 'X:/lua-5.1.5/src'
mingw32-make[2]: 'lua.exe' is up to date.
mingw32-make[2]: Leaving directory 'X:/lua-5.1.5/src'
mingw32-make "LUAC_T=luac.exe" luac.exe
mingw32-make[2]: Entering directory 'X:/lua-5.1.5/src'
mingw32-make[2]: 'luac.exe' is up to date.
mingw32-make[2]: Leaving directory 'X:/lua-5.1.5/src'
mingw32-make[1]: Leaving directory 'X:/lua-5.1.5/src'
cd src && mkdir -p /usr/local/bin /usr/local/include /usr/local/lib /usr/local/man/man1 /usr/local/share/lua/5.1 /usr/local/lib/lua/5.1
命令语法不正确。
Makefile:62: recipe for target 'install' failed
mingw32-make: *** [install] Error 1
```
因为在 Windows 下，mkdir 的多层级路径得用 \ 符号分隔  
如果想要顺利安装，可修改 Lua 源码目录下的 Makefile  
不过在 Windows 下已经生成可用的 lua.exe 和 luac.exe  
没有必要强求完成 `make xxx install` 这一步  
故在此不赘述修改步骤