# 表

## 索引

[Lua 参考手册-值与类型](http://www.runoob.com/manual/lua53doc/manual.html#2.1) 中这样写道：

```text
表是 Lua 中唯一的数据结构，
它可被用于表示普通数组、序列、符号表、集合、记录、图、树等等。
对于记录，Lua 使用域名作为索引。
语言提供了 a.name 这样的语法糖来替代 a["name"] 这种写法以方便记录这种结构的使用。
```

## 表构建

[Lua 参考手册-表构建](http://www.runoob.com/manual/lua53doc/manual.html#3.4.9) 已经写得很详细了  
不过值得注意的是，以下两种有序表的构建方式是有所不同的：

```lua
local foo1 = {
    [1] = 'a',
    [2] = 'b',
    [3] = 'c',
    [4] = 'd',
    [5] = 'e',
}

local foo2 = {
    'a',
    'b',
    'c',
    'd',
    'e',
}
```

不同之处在于，foo1 是哈希表，foo2 才是数组  
看 ltable.c 文件的以下函数：

```c
Table *luaH_new (lua_State *L, int narray, int nhash) {
  Table *t = luaM_new(L, Table);
  luaC_link(L, obj2gco(t), LUA_TTABLE);
  t->metatable = NULL;
  t->flags = cast_byte(~0);
  /* temporary values (kept only if some malloc fails) */
  t->array = NULL;
  t->sizearray = 0;
  t->lsizenode = 0;
  t->node = cast(Node *, dummynode);
  setarrayvector(L, t, narray);
  setnodevector(L, t, nhash);
  return t;
}
```

narray 是数组元素个数，nhash 是非数组元素个数  
当 foo1 创建时 narray 为 0， nhash 为 5  
当 foo2 创建时 narray 为 5， nhash 为 0  
narray 的不同会导致取表长度时，运算过程和结果的不同

## 取长度

[Lua 参考手册-取长度操作符](http://www.runoob.com/manual/lua53doc/manual.html#3.4.7)  
table 的取长度算法主要是 ltable.c 的 luaH\_getn 和 unbound\_search 函数：

```c
static int unbound_search (Table *t, unsigned int j) {
  unsigned int i = j;  /* i is zero or a present index */
  j++;
  /* find `i' and `j' such that i is present and j is not */
  while (!ttisnil(luaH_getnum(t, j))) {
    i = j;
    j *= 2;
    if (j > cast(unsigned int, MAX_INT)) {  /* overflow? */
      /* table was built with bad purposes: resort to linear search */
      i = 1;
      while (!ttisnil(luaH_getnum(t, i))) i++;
      return i - 1;
    }
  }
  /* now do a binary search between them */
  while (j - i > 1) {
    unsigned int m = (i+j)/2;
    if (ttisnil(luaH_getnum(t, m))) j = m;
    else i = m;
  }
  return i;
}

/*
** Try to find a boundary in table `t'. A `boundary' is an integer index
** such that t[i] is non-nil and t[i+1] is nil (and 0 if t[1] is nil).
*/
int luaH_getn (Table *t) {
  unsigned int j = t->sizearray;
  if (j > 0 && ttisnil(&t->array[j - 1])) {
    /* there is a boundary in the array part: (binary) search for it */
    unsigned int i = 0;
    while (j - i > 1) {
      unsigned int m = (i+j)/2;
      if (ttisnil(&t->array[m - 1])) j = m;
      else i = m;
    }
    return i;
  }
  /* else must find a boundary in hash part */
  else if (t->node == dummynode)  /* hash part is empty? */
    return j;  /* that is easy... */
  else return unbound_search(t, j);
}
```

其中 t-&gt;sizearray 就是由前面提到的 narray 赋值  
故以 foo2 为例，`#foo2` 就会直接 return 这个值  
但是按照以上的取长度算法，下面这段 lua 脚本代码的结果就比较特别：

```lua
local foo3 = {
    [1] = 'a',
    [2] = 'b',
    [4] = 'c',
    [8] = 'c',
    [16] = 'c',
}

local foo4 = {
    'a',
    'b',
    'c',
    nil,
    'd',
    'e',
}

print(#foo3);
print(#foo4);
```

结果显示 foo3 的长度为 16，foo4 的长度为 6  
foo3 的长度结果完全由 unbound\_search 函数运算得出  
foo4 的长度结果则由表构建时的 narray 决定

## 有序表的遍历

值得注意的是，虽然 \#foo4 为 6  
但如果执行以下 lua 脚本代码，只会输出 nil 之前的元素：

```lua
for i, v in ipairs(foo4) do
    print(i, v);
end
```

结果为：

```text
1       a
2       b
3       c
```

## 序列化与反序列化库

在各种应用中  
经常会使用到表的序列化与反序列化  
在此简单介绍下一个好用的库 Serpent  
[Serpent 的 GitHub 主页](https://github.com/pkulchenko/serpent)

把 serpent.lua 复制黏贴到 Lua51/script 目录下  
然后在 require 其他模块之前，先键入以下代码：

```lua
serpent = require("serpent");
```

即可随意使用 serpent.lua 定义的函数  
其他详情请参考 [Serpent 的 GitHub 主页](https://github.com/pkulchenko/serpent)

