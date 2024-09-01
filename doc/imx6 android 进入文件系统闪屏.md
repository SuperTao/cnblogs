imx6进入文件系统的时候都会闪屏，应该是framebuffer未初始化，就已经打开了背光。目前解决办法,在kenel阶段关闭背光，显示android的开机动画之后(此时framebuffer已经初始化),再打开背光，这样就看不到闪屏现象,而动画持续时间比较长，所以也能看到开机动画。 

#### 参考链接

　　http://www.iloveandroid.net/2015/09/24/Android_init_1/

　　http://blog.csdn.net/andrewblog/article/details/17122303

#### 文件分析

主要修改ramdisk的init.rc文件，将开机动画的操作进行更改。大致分析记录如下，并附上更改的内容。

```
import /init.usb.rc     # 导入其他脚本文件 
import /init.${ro.hardware}.rc
import /init.trace.rc

on init             # 初始化阶段 , 运行的command 
	chmod 666 /dev/ttymxc0      # command
	chmod 666 /dev/ttymxc1
	chmod 666 /dev/ttymxc2
	chmod 666 /dev/ttymxc3
	chmod 666 /dev/ttymxc4
	chmod 666 /dev/i2c-0
	chmod 666 /dev/i2c-1
	chmod 666 /dev/i2c-2
	chmod 666 /dev/i2c-3
	chmod 666 /dev/buzzer
	chmod 666 /dev/mx6check
	chmod 666 /dev/shutdown
# 设置环境变量
# setup the global environment
    export PATH /sbin:/vendor/bin:/system/sbin:/system/bin:/system/xbin
    export LD_LIBRARY_PATH /vendor/lib:/system/lib
    export ANDROID_BOOTLOGO 1
    export ANDROID_ROOT /system
    export ANDROID_ASSETS /system/app
    export ANDROID_DATA /data
    export ANDROID_STORAGE /storage
    ......
    
    chown system system /sys/class/leds/keyboard-backlight/brightness
    echo 122 > /sys/class/backlight/pwm-backlight.1/brightness      # 背光
    chown system system /sys/class/leds/lcd-backlight/brightness

# Tony 2016-8-29
service bootanim /system/etc/bootanimation.sh # 开启启动画面调用的脚本
    class main
    oneshot
# 原来的bootanim注释掉
#service bootanim /system/bin/bootanimation # 开启启动画面调用的脚本, bootanimation是一个可执行文件，直接在shell中执行，会显示开机动画。
#    class main          # 所属类
#    user system         # 切换用户名
#    group graphics      # 切换组名
#    disabled            # 服务不会自动运行，必须显式地通过start命令来启动。
#    oneshot             # 当此服务退出时不会自动重启.
```

自己实现的脚本，当要显示android动画时调用

bootanimation.sh

```
#!/system/bin/sh

/etc/backlight.sh &      #后台运行，打开背光

#android boot animation
/system/bin/bootanimation   #显示开机动画
```

backlight.sh

```
#!/system/bin/sh

sleep 1 #延时1s再背光，等framebuffer已经初始化，android已经动画已经显示，再开背光。

# 开启2个LVDS的背光
#lvds 0 backlight enable
echo 175 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio175/direction
echo 1   > /sys/class/gpio/gpio175/value
echo 175 > /sys/class/gpio/unexport

#lvds 0 backlight enable
echo 176 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio176/direction
echo 1   > /sys/class/gpio/gpio176/value
echo 176 > /sys/class/gpio/unexport
```

#### Author

Tony Liu

2016-8-29