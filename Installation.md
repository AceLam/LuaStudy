# 编译与安装

[LuaReadMe]: http://www.lua.org/manual/5.3/readme.html
[LuaReadMeCn]: https://cloudwu.github.io/lua53doc/
[DownMinGW]: https://sourceforge.net/projects/mingw/files/latest/download?source=files
[AllLuaVersions]: http://www.lua.org/versions.html
[Lua515]: http://www.lua.org/ftp/lua-5.1.5.tar.gz
[InstallLua]: http://www.runoob.com/lua/lua-environment.html
[LuaVS]: http://blog.csdn.net/linkhai/article/details/45568853


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

主要参考了 [Windows7 下 Lua 的编译和配置][LuaVS]，略作改动

###### 新建解决方案
以 VS 2015 为例  
依次点击菜单 `文件 -> 新建 -> 项目`  
依次选择 `模板 -> 其他项目类型 -> Visual Studio 解决方案`  

名称填 `Lua51`  
位置假设填 `X:`  
点击 `确定`

###### 新建项目

把解压的 Lua 源码的 src 目录复制到 `X:\Lua51` 下  
删除 src 下的 `Makefile` 文件  

在 VS 中，右击 `解决方案资源管理器` 下的解决方案  
依次点击菜单 `添加 -> 新建项目`  
依次选择 `Visual C++ -> 空项目`  

名称填 `lua51`  
点击 `确定`  

按照以上新建项目的步骤再新建两个分别名为 `lua` 和 `luac` 的项目

###### 添加文件

右击 `lua51` 项目，依次点击菜单 `添加->现有项`  
选择 `X:\Lua51\src` 下除了 `lua.c` 和 `luac.c` 外的所有文件  
点击 `添加`  

`lua` 项目添加 `lua.c`  
`luac` 项目添加 `luac.c`  
步骤同上


###### 配置

右击 `lua51` 项目，选择 `属性` 菜单  
依次选择 `配置属性 -> C/C++ -> 预处理器 -> 预处理器定义`  
添加 `_CRT_SECURE_NO_DEPRECATE`  
`lua` 和 `luac` 项目的操作步骤同上  
否则会在生成时报如以下的此类错误：
```
...
错误	C4996	'sprintf': This function or variable may be unsafe. Consider using sprintf_s instead. To disable deprecation, use _CRT_SECURE_NO_WARNINGS. See online help for details.
...
```
右击 `lua51` 项目，`属性 -> 配置属性 -> 常规 -> 配置类型`  
选择 `静态库`  
点击 `应用`  
点击 `确定`  

`lua` 项目，`属性 -> 配置属性 -> 链接器 -> 常规 -> 附加库目录` ，添加 `$(OutDir)`  
`属性 -> 配置属性 -> 链接器 -> 输入 -> 附加依赖项` ，添加 `lua51.lib`  
点击 `应用`  
点击 `确定`  
`luac` 项目同上  

###### 生成

由于 `lua` 和 `luac` 项目依赖 `lua51` 生成的 `lua51.lib`  
所以首先得生成 `lua51` ，再生成另两个项目  

所有项目都生成完成后，会在输出目录下输出解释器 lua.exe 和编译器 luac.exe


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
否则会在 make 时报以下错误：
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
  
此时，若前面没有修改 `src/Makefile` 则会报以下错误：
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