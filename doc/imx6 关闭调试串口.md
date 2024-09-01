需要关闭imx6调试串口，用作普通的串口使用。

参考链接

http://blog.csdn.net/neiloid/article/details/7585876

http://www.cnblogs.com/helloworldtoyou/p/5437867.html

更改kernel中配置， make menuconfig
```
Symbol: SERIAL_IMX_CONSOLE [=y]             
Type  : boolean                             
Prompt: Console on IMX serial port          
    Defined at drivers/tty/serial/Kconfig:826 
    Depends on: HAS_IOMEM [=y] && SERIAL_IMX [=y]
    Location:                                    
      -> Device Drivers                          
        -> Character devices                     
          -> Serial drivers                      
            -> IMX serial port support (SERIAL_IMX [=y])
```

Tony Liu

2016-12-23, Shenzhen