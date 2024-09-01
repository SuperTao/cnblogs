新的项目中使用到了电池。电池的guage使用TI的bq20z75。kernel的驱动中已经有bq20z75的驱动，只要稍加修改就可以使用。

#### 参考链接

http://www.ti.com/lit/er/sluu265a/sluu265a.pdf

http://www.ti.com/lit/an/slua421a/slua421a.pdf

http://www.deyisupport.com/question_answer/analog/battery_management/f/35/t/77397.aspx

https://www.element14.com/community/thread/26389/l/bus-speed-of-devi2c-2-on-imx6q?displayFullThread=true


#### driver

驱动源码位于 kernel/drivers/power/bq20z75.c

内核中添加驱动。
```
CONFIG_BATTERY_BQ20Z75:     

   Say Y to include support for TI BQ20z75 SBS-compliant    
   gas gauge and protection IC.       
                                      
   Symbol: BATTERY_BQ20Z75 [=y]       
   Type  : tristate                   
   Prompt: TI BQ20z75 gas gauge       
     Defined at drivers/power/Kconfig:118      
     Depends on: POWER_SUPPLY [=y] && I2C [=y]  
     Location:                                  
       -> Device Drivers                        
         -> Power supply class support (POWER_SUPPLY [=y]) 
```

调试的时候最好编译成模块。

#### device

kernel/arch/arm/mach-mx6/board-mx6q_sabresd.c

在对应的i2c总线上添加设备。
```
static struct i2c_board_info mxc_i2c2_board_info[] __initdata = {
{
	...
	{
		I2C_BOARD_INFO("bq20z75", 0xb),
	},
	...
};
```

datasheet里面说地址是0x16,但是我发送地址0x16的时候，使用逻辑分析仪查看数据，并没有回应。查看资料有些说法是，芯片seal了，需要unseal。但是现在连地址都没有回应，所以可能是设备地址错了。

所以写了程序将i2c中线上的所有的地址都试了一遍，发现地址是0xb的时候有回应。所以设备的地址应该是0xb。

#### rootfs

imx6 添加bq20z75.

重新编译内核,驱动加载。

加载了驱动之后，会在/sys/class/power_supply生成battery目录，如下：

```
root@freescale /sys/class/power_supply$ ls
battery
root@freescale /sys/class/power_supply$ cd battery
root@freescale /sys/class/power_supply/battery$ ls
capacity            device              present             time_to_empty_avg
charge_full         energy_full         serial_number       time_to_full_avg
charge_full_design  energy_full_design  status              type
charge_now          energy_now          subsystem           uevent
current_now         health              technology          voltage_max_design
cycle_count         power               temp                voltage_now
root@freescale /sys/class/power_supply/battery$ cat temp 
235
root@freescale /sys/class/power_supply/battery$ cat voltage_now 
11822000
```

Tony Liu

2016-12-3, Shenzhen