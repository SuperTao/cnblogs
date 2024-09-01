编译内核时，使用默认的配置进行编译。出现错误:undefined reference to `switch_dev_unregister',undefined reference to `switch_set_state'.

参考链接：

　　http://www.cnblogs.com/zengjfgit/p/4882146.html

make imx_defconfig

make uImage

编译出错。

```
drivers/built-in.o: In function `mxc_hdmi_remove':
clkdev.c:(.text+0x12610): undefined reference to `switch_dev_unregister'
clkdev.c:(.text+0x1261c): undefined reference to `switch_dev_unregister'
drivers/built-in.o: In function `hotplug_worker':
clkdev.c:(.text+0x14424): undefined reference to `switch_set_state'
clkdev.c:(.text+0x14434): undefined reference to `switch_set_state'
clkdev.c:(.text+0x1462c): undefined reference to `switch_set_state'
clkdev.c:(.text+0x1463c): undefined reference to `switch_set_state'
drivers/built-in.o: In function `mxc_hdmi_probe':
clkdev.c:(.devinit.text+0x3c8): undefined reference to `switch_dev_register'
clkdev.c:(.devinit.text+0x3d4): undefined reference to `switch_dev_register'
sound/built-in.o: In function `usb_audio_disconnect':
last.c:(.text+0x16954): undefined reference to `switch_set_state'
last.c:(.text+0x1695c): undefined reference to `switch_dev_unregister'
sound/built-in.o: In function `usb_audio_probe':
last.c:(.text+0x16db8): undefined reference to `switch_dev_register'
last.c:(.text+0x16e3c): undefined reference to `switch_set_state'
last.c:(.text+0x16f18): undefined reference to `switch_dev_register'
sound/built-in.o: In function `hp_jack_status_check':
last.c:(.text+0x364dc): undefined reference to `switch_set_state'
last.c:(.text+0x36570): undefined reference to `switch_set_state'
sound/built-in.o: In function `imx_wm8962_remove':
last.c:(.devexit.text+0x2b8): undefined reference to `switch_dev_unregister'
sound/built-in.o: In function `imx_wm8962_probe':
last.c:(.devinit.text+0x6e4): undefined reference to `switch_dev_register'
last.c:(.devinit.text+0x718): undefined reference to `switch_set_state'
make: *** [.tmp_vmlinux1] Error 1
cp: cannot stat `arch/arm/boot/uImage': No such file or directory
```

解决方法：
```
打开switch选项

make menuconfig
    Device Drivers  --->  
	<*> Switch class support  --->   
        --- Switch class support   
		<*>   GPIO Swith support  
```

Tony Liu

2016-11-19, Shenzhen