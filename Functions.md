# 函数

## 可变参数

值得注意的是，arg 和 ... 不能在同一函数内使用  
否则会报 arg 为 nil 的错  
可变参数的各种使用例子的代码和输出结果如下：
```Lua
local function vargs1(...)
    print(select("#", ...)); --> 3
    print(select(2, ...)); --> 5       6

    local args = {...};
    print(#args); --> 3
    print(args[1]); --> 4
    print(serpent.dump(args)); --> do local _={4,5,6};return _;end
end

local function vargs2(...)
    print(arg["n"]); --> 3
    print(#arg); --> 3
    print(arg[1]); --> 7
    print(serpent.dump(arg)); --> do local _={[1]=7,[2]=8,[3]=9,n=3};return _;end
end

vargs1(4, 5, 6);
vargs2(7, 8, 9);
```