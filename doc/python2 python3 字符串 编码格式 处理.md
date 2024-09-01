使用python的ctypes调用c语言中的函数，传入字符串，打印输出异常。解决方法记录于此。

参考连接：

http://blog.csdn.net/u011546806/article/details/44936303

主要原因是编码格式不同导致的。python2和python3采用的默认编码不同。

python2默认就是str和unicode，str和unicode可以直接进行连接。python3默认的字符串编码是bytes和str。如果要操作unicode格式的，需要通过encode()函数先转换。

c文件如下。

test.c

```
#include <stdio.h>

int test(char *temp)
{
    printf("temp:%s\n", temp);
    return 0;
}
```

test.py

```
#!/usr/bin/env python
import test
import os

from ctypes import *

test = cdll.LoadLibrary(os.getcwd() + '/libtest.so')
test.test('hello')
```

python2运行脚本输出如下

```
qt@tony:/mnt/hgfs/Desktop/$ /usr/bin/python2 test.py
temp:hello
temp:hello
```

python3运行结果

```
qt@tony:/mnt/hgfs/Desktop/$ /usr/bin/python3 test.py
temp:h
temp:h
```

解决方法，使用encode('utf-8')将字符串转化成unicode格式.

更改如下

```
#!/usr/bin/env python
import test
import os

from ctypes import *

test = cdll.LoadLibrary(os.getcwd() + '/libtest.so')
# 将str转换成unicode
test.test('hello'.encode('utf-8'))
```

Tony Liu

2017-6-2, Shenzhen