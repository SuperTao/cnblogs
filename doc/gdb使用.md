
#### 参考链接

https://www.yanbinghu.com/2019/04/20/41283.html

#### 编译代码

调试代码hello.c如下。

```
#include <stdio.h>

int add(int a, int b)
{
        int c = a + b;
        return c;
}

int main(void)
{
        int i, ret;

        ret = add(7, 8);
        printf("7 add 8 equals %d\n", ret);

        for (i = 0; i < 10; i++)
        {
                printf("hello world\n");
        }
        return 0;
}
```

要使用gdb调试程序，需要在编译时添加`-g`选项。

`gdb -g -o hello hello.c`

#### 启动gdb

`gdb hello`

```
GNU gdb (Ubuntu 8.1-0ubuntu3) 8.1.0.20180409-git
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from hello...done.
(gdb)
```

#### 查看代码

* list

默认会显示10行代码，可以通过命令设置，例如要设置成20行。

`set listsize 20`

还可以通过list + 行号，列出某一行前后的的代码。

`list 9`

查看指定行之间的源码

`list first,last`

#### 运行代码

`run`

#### 设置断点

* 在第13行设置断点

`break 13`

* 查看断点信息

`info breakpoints`

* 删除断点

```
disable       禁用所有断点
disable num   禁用第num行的断点

clear         删除breakpoints

delete         删除断点breakpoints, watchpoints, catchpoints
delete num     删除第num行的断点
```

#### 打印变量的值

打印变量i的值

`print i`

按格式打印

`p/x i`

```
x 按十六进制格式显示变量。
d 按十进制格式显示变量。
u 按十六进制格式显示无符号整型。
o 按八进制格式显示变量。
t 按二进制格式显示变量。
a 按十六进制格式显示变量。
c 按字符格式显示变量。
f 按浮点数格式显示变量。
```

#### 单步运行

* next

遇到断点停止时，使用next命令继续运行

* step

进入子函数运行

* continue

继续运行，直到遇到下一个断点

#### 自动显示变量内容

`display i`

查看自动显示的变量

`info display`

清除显示变量

`delete display i`

去使能

`disable display i`

#### 查看内存值

`x /nfu <addr>`

```
说明
x 是 examine 的缩写

n表示要显示的内存单元的个数

f表示显示方式, 可取如下值
x 按十六进制格式显示变量。
d 按十进制格式显示变量。
u 按十进制格式显示无符号整型。
o 按八进制格式显示变量。
t 按二进制格式显示变量。
a 按十六进制格式显示变量。
i 指令地址格式
c 按字符格式显示变量。
f 按浮点数格式显示变量。

u表示一个地址单元的长度
b表示单字节，
h表示双字节，
w表示四字节，
g表示八字节
```
