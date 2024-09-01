有时候需要将开机启动的信息输出到LCD上，并且在终端上进行调试。本文记录更改的方法。

#### 参考链接

http://blog.csdn.net/chenbang110/article/details/7870072

https://e2e.ti.com/support/embedded/linux/f/354/t/324198

https://blackfin.uclinux.org/doku.php?id=linux-kernel:drivers:fbcon

#### uboot

uboot bootargs添加console=tty0

如下，debug信息即会在lcd输出，也会输出到LCD。

setenv bootargs console=tty0 console=ttymxc0,115200

#### kernel

kernel打开配置, uboot配置之后，开机启动信息就会在lcd上显示。

make menuconfig
```
CONFIG_FRAMEBUFFER_CONSOLE
CONFIG_FRAMEBUFFER_CONSOLE_DETECT_PRIMARY

Location: 
     -> Device Drivers  
          -> Graphics support  
            -> Console display driver support
                <*> Framebuffer Console support  
                [*]   Map the console to the primary display device
```

#### rootfs

/etc/inittab 添加, 这样就能够将终端在串口和LCD上输出。
```
tty0::respawn:-/bin/sh		# 在LCD上显示
ttymxc0::respawn:-/bin/sh	# 在串口显示
```

Tony Liu

2016-12-4, Shenzhen