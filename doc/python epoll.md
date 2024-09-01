需要用python实现中断的功能，所以用epoll监听gpio文件的变化。写个demo测试一下。

参考：

http://www.cnblogs.com/coser/archive/2012/01/06/2315216.html

http://www.cnblogs.com/helloworldtoyou/p/5532271.html

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import select

import gpio
# 申请gpio
def gpioExport(gpioIndex):
    with open('/sys/class/gpio/export', 'wb') as f:
        f.write(str(gpioIndex).encode())
# 设置gpio为输入
def setInput(gpioIndex):
    with open('/sys/class/gpio/gpio%d/direction' % gpioIndex, 'wb') as f:
        f.write('in'.encode())
# 设置gpio触发方式
def setEdge(gpioIndex, edge):
    with open('/sys/class/gpio/gpio%d/edge' % gpioIndex, 'wb') as f:
        f.write(edge.encode())

gpioExport(15)
gpioExport(36)

setInput(15)
setInput(36)
# 上升沿，下降沿触发
setEdge(15, 'both')
setEdge(36, 'both')
# 这里和C语言中返回值不同。C语言返回值是个int，表示文件描述符的值，而python中需要通过fileno()函数来获取。
# 读取值
f1 = open('/sys/class/gpio/gpio%d/value' % 15, 'r+')
f2 = open('/sys/class/gpio/gpio%d/value' % 36, 'r+')

print ('fd1: %d' % f1.fileno())
print ('fd2: %d' % f2.fileno())

epoll = select.epoll()

# 注册
epoll.register(f1, select.EPOLLERR | select.EPOLLPRI)
epoll.register(f2, select.EPOLLERR | select.EPOLLPRI)

# 进入主循环，监听gpio的电平变化，
while True:
    events = epoll.poll()	# 没有设置时间，一直阻塞
    for fileno, event in events:
		# 有数据需要读取
        if event & select.EPOLLPRI:
	    # f1发生变化
            if fileno == f1.fileno():
                print ('f1: %s' % f1.read().strip('\n'))
                # 文件读取完，将文件指针偏移到文件开头，方便下次读取
                f1.seek(0, 0)
	    # f2发生变化
            elif fileno == f2.fileno():
                print ('f2: %s' % f2.read().strip('\n'))
                f2.seek(0, 0)
		# 出现错误
        if event & select.EPOLLERR:
                pass
```

Tony Liu

2017-6-7, Shenzhen