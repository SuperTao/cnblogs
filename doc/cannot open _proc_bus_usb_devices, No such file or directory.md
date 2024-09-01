由于kernel config中没有打开对应的配置。

```
make menuconfig
选择：
     Device Drivers  --->
	    [*] USB support  --->   
	          [*]     USB device filesystem (DEPRECATED)   
                  [*]     USB device class-devices (DEPRECATED)  

    USB device filesystem：        会生成 /proc/bus/usb
    USB device class-devices：     生成 /dev/usbdev1.1等节点。 
    DEPRECATED表示不赞成，那么以后的内核可能就不再支持.
    目前使用的Linux内核版本是3.0.35
```

Tony Liu

2016-11-19, Shenzhen