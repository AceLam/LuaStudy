# 运行 Lua 脚本代码的方式

## 命令行输入代码运行

直接双击 lua.exe  
或者在命令行模式下  
执行命令：
```
$ lua
```
命令行窗口会有如下内容：
```
Lua 5.1.5  Copyright (C) 1994-2012 Lua.org, PUC-Rio
>
```
输入 `print("hello lua")` 并回车  
则命令行窗口会有如下内容：
```
Lua 5.1.5  Copyright (C) 1994-2012 Lua.org, PUC-Rio
> print("hello lua")
hello lua
>
```

## 运行 lua 文件

创建 main.lua 文件  
其中的代码如下：
```
print("hello lua")
```
在命令行模式下  
执行命令：
```
$ lua main.lua
```
命令行窗口会有如下内容：
```
hello lua
```