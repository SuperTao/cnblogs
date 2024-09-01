通过调试口查看树莓派开机启动信息,学习python控制串口的方法。

##### 参考链接：

　　http://www.elinux.org/Serial_port_programming

##### 硬件连接：

- 硬件原理图链接： [raspberrypi-B-Plus-V1.2](https://www.raspberrypi.org/documentation/hardware/raspberrypi/schematics/Raspberry-Pi-B-Plus-V1.2-Schematics.pdf)

- 通过USB转ttl转接板与树莓派的TXD,RXD,GND连接。

- 由于树莓派的输出是3.3V，所以usb转ttl的转接板也需要调节为3.3V输出。

- 调试串口参数：115200，8 N 1

##### python串口包安装

　　sudo apt-get install python-serial

##### python调试程序：

```
import serial

#port = serial.Serial("/dev/ttyAMA0", baudrate=115200, timeout=3.0) #接收超时3s
port = serial.Serial("/dev/ttyAMA0", baudrate=115200)               #一直阻塞

while True:
    port.write("\r\nSay something:")
    #rcv = port.read(10)                        #读取10个字符，读满10个才结束
    rcv = port.readline()                       #读取一行数据，遇到'\n'结束
    port.write("\r\nYou sent:" + repr(rcv))     #收到的数据发送到串口 
```
##### Author

Tony Liu

2016-8-28, Shenzhen