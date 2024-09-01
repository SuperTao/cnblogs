在使用GPIO的时候，有时候不知道GPIO的状态，也不知道在内核中GPIO是否申请成功。

可以通过/sys/kernel/debug/gpio这个文件来查看。这个文件显示了申请成功的GPIO的输入输出状态和电平。

#### 参考

http://elinux.org/GPIO

https://developer.ridgerun.com/wiki/index.php/How_to_use_GPIO_signals

#### 配置

* 内核打开debugfs支持

```
Symbol: DEBUG_FS [=y]
   Prompt: Debug Filesystem
     Defined at lib/Kconfig.debug:77
     Depends on: SYSFS     
     Location:
       -> Kernel configuration
         -> Kernel hacking      
```

* 挂载debugfs
	 
`mount -t debugfs none /sys/kernel/debug`
		 
#### 测试

```
root@android:/data # cat /sys/kernel/debug/gpio                                      
GPIOs 0-31, gpio-0:
 gpio-0   (ESDHC_CD            ) in  lo
 gpio-4   (btn volume-up       ) in  hi
 gpio-5   (btn volume-down     ) in  hi
 gpio-22  (AD7606_STBY         ) out lo
 gpio-31  (AD7606_CONVST       ) out lo

GPIOs 32-63, gpio-1:
 gpio-58  (spi_imx             ) out lo
 gpio-59  (spi_imx             ) in  lo
 gpio-60  (sysfs               ) out lo

GPIOs 64-95, gpio-2:
 gpio-83  (sensor pwr en       ) out lo
 gpio-86  (usb-pwr             ) out lo
 gpio-93  (btn power-key       ) in  hi

GPIOs 96-127, gpio-3:
 gpio-102 (matrix_kbd_col      ) out lo
 gpio-103 (matrix_kbd_row      ) in  hi
 gpio-104 (matrix_kbd_col      ) out lo
 gpio-105 (matrix_kbd_row      ) in  hi
 gpio-106 (matrix_kbd_col      ) out lo
 gpio-107 (matrix_kbd_row      ) in  hi
 gpio-108 (matrix_kbd_col      ) out lo
 gpio-109 (matrix_kbd_row      ) in  hi
 gpio-110 (scl                 ) in  hi
 gpio-111 (sda                 ) in  hi

GPIOs 128-159, gpio-4:

GPIOs 160-191, gpio-5:
 gpio-167 (AD7606_OS1          ) out lo
 gpio-168 (AD7606_OS0          ) out lo
 gpio-169 (AD7606_OS2          ) out lo
 gpio-170 (AD7606_RESET        ) out lo
 gpio-175 (cabc-en0            ) out lo
 gpio-176 (cabc-en1            ) out lo

GPIOs 192-223, gpio-6:
 gpio-192 (usb-h1-pwr          ) out lo
 gpio-205 (pFUZE-int           ) in  hi
```

Tony Liu

2017-1-13, Shenzhen