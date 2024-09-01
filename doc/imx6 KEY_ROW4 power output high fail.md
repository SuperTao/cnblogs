imx6 KEY_ROW4的pin设置成gpio之后，不能够输出高电平。解决方法记录于此。

参考链接：

https://lists.yoctoproject.org/pipermail/meta-freescale/2014-January/006271.html


解决方法
```
打开board-mx6dl_sabresd.h

// Tony 2016-11-28,添加下面的宏
#define MX6DL_ENET_PAD_CTRL_PD (PAD_CTL_PKE | PAD_CTL_PUE  |        \
        PAD_CTL_PUS_100K_DOWN | PAD_CTL_SPEED_MED |     \
        PAD_CTL_DSE_40ohm   | PAD_CTL_HYS)

static iomux_v3_cfg_t mx6dl_sabresd_pads[] = {
	......
	//注释默认的配置
	//MX6DL_PAD_KEY_ROW4__GPIO_4_15,      // POWER
	//更改为
	IOMUX_PAD(0x0650, 0x0268, 5, 0x0000, 0, MX6DL_ENET_PAD_CTRL_PD),
	......
};


默认的使用的是NO_PAD_CTRL,如下：
arch/arm/plat-mxc/include/mach/iomux-mx6dl.h
#define MX6DL_PAD_KEY_ROW4__GPIO_4_15                                          \
         IOMUX_PAD(0x0650, 0x0268, 5, 0x0000, 0, NO_PAD_CTRL)
```

Tony Liu

2016-11-28, Shenzhen