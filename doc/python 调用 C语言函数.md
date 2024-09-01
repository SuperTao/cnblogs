python可以直接调用C语言的函数,本文记录用ctypes调用c语言的方法。

test.c

```
#include <stdio.h>

int test(char *temp)
{
    printf("temp:%s\n", temp);
    return 0;
}
```

编译成动态库 

`gcc test.c -fPIC -shared -o libtest.so`


test.py

```
#!/usr/bin/env python
import os

from ctypes import *
# 加载动态库
t = cdll.LoadLibrary(os.getcwd() + '/libtest.so')
# 调用其中的函数
t.test('hello'.encode())
```

运行结果

```
qt@tony:~$ ./test.py
temp:hello
```

Tony Liu

2017-6-2, Shenzhen