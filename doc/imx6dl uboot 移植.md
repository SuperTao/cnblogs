新版的BSP引进的设备树的机制，在uboot中还添加了menuconfig的配置菜单。

参考官网的文档进行uboot移植，本文使用的cpu是imx6dl，uboot版本2015.04。

我要添加一个名称是mx6sabresd_sbc的板子，具体操作如下：

1.添加目录目录

`cp board/freescale/mx6sabresd board/freescale/mx6sabresd_sbc -r`

2.添加参数配置文件

`cp include/configs/mx6sabresd.h include/configs/mx6sabresd_sbc.h`

3.复制编译配置文件

创建新的文件configs/mx6dlsabresdandroid_sbc_defconfig

`cp configs/mx6dlsabresdandroid_defconfig configs/mx6dlsabresdandroid_sbc_defconfig`

更改内容如下

```
CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/freescale/mx6sabresd_sbc/mx6dlsabresd_sbc.cfg,MX6DL,SYS_USE_SPINOR,ANDROID_SUPPORT"
CONFIG_ARM=y
CONFIG_TARGET_MX6SABRESD_SBC=y
CONFIG_SYS_MALLOC_F=y
CONFIG_SYS_MALLOC_F_LEN=0x400
CONFIG_DM=y
CONFIG_DM_THERMAL=y
```

将mx6dlsabresd.cfg更改为mx6dlsabresd_sbc.cfg.

4.文件重命名

将对应的文件从命名

`cp board/freescale/mx6sabresd_sbc/mx6sabresd.c board/freescale/mx6sabresd_sbc.c`

`cp board/freescale/mx6sabresd_sbc/mx6dlsabresd.cfg board/freescale/mx6dlsabresd_sbc.cfg`

5.更改板子的Makefile

更改board/freescale/mx6sabresd_sbc/Makefile

```
obj-y  := mx6sabresd_sbc.o

extra-$(CONFIG_USE_PLUGIN) :=  plugin.bin
$(obj)/plugin.bin: $(obj)/plugin.o
        $(OBJCOPY) -O binary --gap-fill 0xff $< $@
```

6.更改Kconfig

更改board/freescale/mx6sabresd_sbc/Kconfig

```
if TARGET_MX6SABRESD_SBC

config SYS_BOARD
        default "mx6sabresd_sbc"

config SYS_VENDOR
        default "freescale"

config SYS_SOC
        default "mx6"

config SYS_CONFIG_NAME
        default "mx6sabresd_sbc"

endif
```

更改arch/arm/Kconfig

添加下面的内容
```
config TARGET_MX6SABRESD_SBC
        bool "Support mx6sabresd_sbc"
        select CPU_V7
		
source "board/freescale/mx6sabresd_sbc/Kconfig"
```

7.重新编译uboot

```
export ARCH=arm

export CROSS_COMPILE=~/myandroid/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin/arm-eabi-

make distclean

#For i.MX 6DualLite SABRE-SD:
make mx6dlsabresdandroid_sbc_config

make
```

Tony Liu

2017-3-12, Shenzhen