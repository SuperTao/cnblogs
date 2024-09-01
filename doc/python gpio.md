打算用python控制gpio，网上找的都是一些关于树莓派如何操作gpio的文档，只针对树莓派。所以就自己封装一下函数，方便以后使用。在linux上已经生成了和gpio相关的文件，只要对文件进行读取即可。

参考:

https://www.kernel.org/doc/Documentation/gpio/sysfs.txt

https://coldnew.github.io/f7349436/

在查看这个之前需要对linux文件系统对gpio控制有基本的了解。可以阅读参考文档。

这里封装了简单的函数。

```
#!/usr/bin/env python

import io 

gpioValue = {'high': 1, 'low': 0}

# gpio的几种触发方式
NONE    = 'none'
RISING  = 'rising'
FALLING = 'falling'
BOTH    = 'both'

# 导出gpio,相当于申请，这样其他程序就不能使用这个gpio
def gpioExport(gpioIndex):
    with open('/sys/class/gpio/export', 'wb') as f:
        f.write(str(gpioIndex).encode())

# 释放gpio，释放之后其他程序才能调用这个gpio口 
def gpioUnexport(gpioIndex):
    with open('/sys/class/gpio/unexport', 'wb') as f:
        f.write(str(gpioIndex).encode())
		
# 设置输入
def setInput(gpioIndex):
    with open('/sys/class/gpio/gpio%d/direction' % gpioIndex, 'wb') as f:
        f.write('in'.encode())
		
# 设置输出
def setOutput(gpioIndex):
    with open('/sys/class/gpio/gpio%d/direction' % gpioIndex, 'wb') as f:
        f.write('out'.encode())
		
# 读取输入值
def getInputValue(gpioIndex):
    with open('/sys/class/gpio/gpio%d/value' % gpioIndex, 'r+') as f:
        return f.read().strip('\n')  # delete the '\n' 
		
# 设置输出的值
def setOutputValue(gpioIndex, value):
    with open('/sys/class/gpio/gpio%d/value' % gpioIndex, 'wb') as f:
        return f.write(str(value).encode())
		
# 设置gpio触发方式
def setEdge(gpioIndex, edge):
    with open('/sys/class/gpio/gpio%d/edge' % gpioIndex, 'wb') as f:
        f.write(edge.encode())
		
# 读取gpio的触发方式
def getEdge(gpioIndex):
    with open('/sys/class/gpio/gpio%d/edge' % gpioIndex, 'r+') as f:
        return f.read().strip('\n')
		
# 读取gpio的值
def getGpioValue(gpioIndex):
    gpioExport(gpioIndex)
    setInput(gpioIndex)
    val = getInputValue(gpioIndex)
    gpioUnexport(gpioIndex)
    return val
	
# 获取一组GPIO的值，出入的参数是tuple或者list
# 返回list
def getGpioValues(gpioTuple):
    gpioValues = []
    for i in gpioTuple:
        gpioExport(i)
        setInput(i)
        gpioValues.append(getInputValue(i))
        gpioUnexport(i)
    return gpioValues
# 等待gpio触发事件（还未实现）
def waitForEdge(gpioIndex):
    pass
```

Tony Liu

2017-6-3, Shenzhen