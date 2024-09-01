使用threading模块,写一个的关于python多线程的demo。

参考：

http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386832360548a6491f20c62d427287739fcfa5d5be1f000

http://www.runoob.com/python/python-multithreading.html

http://www.cnblogs.com/fnng/p/3670789.html

实现的方法有很多，这里是继承了threading类。

thread_test.py

```
#!/usr/bin/env python

import threading
from time import ctime,sleep

class music(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)

    def run(self):
        while True:
            print ('music')
            sleep(1)

class move(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)

    def run(self):
        while True:
            print ('move')
            sleep(1)

t1 = music()
t1.start()

t2 = move()
t2.start()
```

运行结果
```
qt@tony:~$ ./thread_test.py 
music
move
music
move
music
move
music
move
music
move
music
move
...
...
```

Tony Liu

2017-6-2, Shenzhen