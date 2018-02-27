# VS 调试 Lua 源码

[Breakpoint]: /assets/breakpoint.png
[Hit]: /assets/hit.png

## 命令行输入代码运行

利用 VS 编译完 Lua 源码之后  
即可运行 lua 工程进行断点调试  
  
设置断点：
![断点][Breakpoint]
运行并输入：
```
print("hello lua")
```
回车后即命中断点：
![命中][Hit]


## 运行 lua 文件

在 `X:\Lua51` 目录下创建名为 script 的目录  
在 script 目录下创建 main.lua 文件  
其中的代码如下：
```
print("hello lua")
```
VS 工程中
`lua` 项目，`属性 -> 配置属性 -> 调试 -> 命令参数` ，添加 `$(SolutionDir)script\main.lua`  
仍然在 `luaB_print` 函数中设置断点  
运行即命中断点