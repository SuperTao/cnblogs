imx6dl需要支持lcd接口的屏，imx6dl的datasheet并没有明确的说明lcd相关的配置，只在Display Content Integrity Checker (DCIC)一章中介绍。本文记录imx6支持lcd的方法。

#### 参考链接

http://developer.toradex.com/knowledge-base/display-output-resolution-and-timings-linux

https://community.nxp.com/thread/307613

http://cache.freescale.com/files/32bit/doc/user_guide/MX53UG.pdf

https://boundarydevices.com/configuring-i-mx6-machines-different-screens-nitrogen6x-sabre-lite/

### uboot

更改bootargs，添加lcd支持：video=mxcfb0:dev=lcd,SEIKO-WVGA,if=RGB24,bpp=32

include/configs/mx6dl_sabresd_android.h

```c
#define CONFIG_EXTRA_ENV_SETTINGS                   \
                "netdev=eth0\0"                                         \
                "ethprime=FEC0\0"                                       \
                "uboot=u-boot.bin\0"                    \
                "kernel=uImage\0"                               \
                "nfsroot=/opt/eldk/arm\0"                               \
                "bootargs_base=setenv bootargs console=ttymxc0,115200 video=mxcfb0:dev=lcd,SEIKO-WVGA,if=RGB24,bpp=32\0"\
                "bootargs_nfs=setenv bootargs ${bootargs} root=/dev/nfs "\
                        "ip=dhcp nfsroot=${serverip}:${nfsroot},v3,tcp\0"\
                "bootcmd_net=run bootargs_base bootargs_nfs; "          \
                        "tftpboot ${loadaddr} ${kernel}; bootm\0"       \
                "bootargs_mmc=setenv bootargs ${bootargs} ip=none "     \
                        "root=/dev/mmcblk0p1 rootwait\0"                \
                "bootcmd_mmc=run bootargs_base bootargs_mmc; "   \
                "mmc dev 2; "   \
                "mmc read ${loadaddr} 0x800 0x3000; bootm\0"    \
                "bootcmd=run bootcmd_mmc\0 "   \
        "splashimage=0x30000000\0"              \
        "splashpos=m,m\0"                   \
        "lvds_num=1\0"  
```
		
### kernel		

需要注册设备和驱动

#### device

配置引脚复用

arch/arm/mach-mx6/board-mx6dl_sabresd.h
```c
static iomux_v3_cfg_t mx6dl_sabresd_pads[] = {
    ......
    MX6DL_PAD_DI0_DISP_CLK__IPU1_DI0_DISP_CLK,
    MX6DL_PAD_DI0_PIN15__IPU1_DI0_PIN15,        /* DE */
    MX6DL_PAD_DI0_PIN2__IPU1_DI0_PIN2,          /* HSync */
    MX6DL_PAD_DI0_PIN3__IPU1_DI0_PIN3,          /* VSync */
    MX6DL_PAD_DI0_PIN4__IPU1_DI0_PIN4,          /* Contrast */
    MX6DL_PAD_DISP0_DAT0__IPU1_DISP0_DAT_0,
    MX6DL_PAD_DISP0_DAT1__IPU1_DISP0_DAT_1,
    MX6DL_PAD_DISP0_DAT2__IPU1_DISP0_DAT_2,
    MX6DL_PAD_DISP0_DAT3__IPU1_DISP0_DAT_3,
    MX6DL_PAD_DISP0_DAT4__IPU1_DISP0_DAT_4,
    MX6DL_PAD_DISP0_DAT5__IPU1_DISP0_DAT_5,
    MX6DL_PAD_DISP0_DAT6__IPU1_DISP0_DAT_6,
    MX6DL_PAD_DISP0_DAT7__IPU1_DISP0_DAT_7,
    MX6DL_PAD_DISP0_DAT8__IPU1_DISP0_DAT_8,
    MX6DL_PAD_DISP0_DAT9__IPU1_DISP0_DAT_9,
    MX6DL_PAD_DISP0_DAT10__IPU1_DISP0_DAT_10,
    MX6DL_PAD_DISP0_DAT11__IPU1_DISP0_DAT_11,
    MX6DL_PAD_DISP0_DAT12__IPU1_DISP0_DAT_12,
    MX6DL_PAD_DISP0_DAT13__IPU1_DISP0_DAT_13,
    MX6DL_PAD_DISP0_DAT14__IPU1_DISP0_DAT_14,
    MX6DL_PAD_DISP0_DAT15__IPU1_DISP0_DAT_15,
    MX6DL_PAD_DISP0_DAT16__IPU1_DISP0_DAT_16,
    MX6DL_PAD_DISP0_DAT17__IPU1_DISP0_DAT_17,
    MX6DL_PAD_DISP0_DAT18__IPU1_DISP0_DAT_18,
    MX6DL_PAD_DISP0_DAT19__IPU1_DISP0_DAT_19,
    MX6DL_PAD_DISP0_DAT20__IPU1_DISP0_DAT_20,
    MX6DL_PAD_DISP0_DAT21__IPU1_DISP0_DAT_21,
    MX6DL_PAD_DISP0_DAT22__IPU1_DISP0_DAT_22,
    MX6DL_PAD_DISP0_DAT23__IPU1_DISP0_DAT_23,
    ......
};
```

设备初始化

arch/arm/mach-mx6/board-mx6q_sabresd.c

```c
// 与bootargs中"dev=lcd,SEIKO-WVGA" 对应 
static struct ipuv3_fb_platform_data sabresd_fb_data[] = {
    // Tony 2016-11-22
    {
    .disp_dev = "lcd",
    .interface_pix_fmt = IPU_PIX_FMT_RGB24,
    .mode_str = "SEIKO-WVGA",
    .default_bpp = 32,
    .int_clk = false,
    },   
    { /*fb0*/
    .disp_dev = "ldb",
    .interface_pix_fmt = IPU_PIX_FMT_RGB24,
    .mode_str = "LDB-XGA",
    //.default_bpp = 16,
    .default_bpp = 24,
    .int_clk = false,
    .late_init = false,
    }, { 
    .disp_dev = "hdmi",
    .interface_pix_fmt = IPU_PIX_FMT_RGB24,
    .mode_str = "1920x1080M@60",
    .default_bpp = 32,
    .int_clk = false,
    .late_init = false,
    }, {
};

static struct fsl_mxc_lcd_platform_data lcdif_data = {
    .ipu_id = 0,		// ipu 0
    .disp_id = 0,		// 第0个接口
    .default_ifmt = IPU_PIX_FMT_RGB24,
};

// lcd初始化
static void __init mx6_sabresd_board_init(void)
{
	......
	imx6q_add_ipuv3(0, &ipu_data[0]);
    if (cpu_is_mx6q()) {
        imx6q_add_ipuv3(1, &ipu_data[1]);
        for (i = 0; i < 4 && i < ARRAY_SIZE(sabresd_fb_data); i++) 
            imx6q_add_ipuv3fb(i, &sabresd_fb_data[i]);
    } else 
        for (i = 0; i < 2 && i < ARRAY_SIZE(sabresd_fb_data); i++) //不知道这里为什么要i<2,难道因为imx6的ipu只有2个接口？
            imx6q_add_ipuv3fb(i, &sabresd_fb_data[i]);

    imx6q_add_lcdif(&lcdif_data);
	......
}
```

#### driver

添加lcd的驱动支持

make menuconfig
选择 SEIKO WVGA Panel 
```
Prompt: SEIKO WVGA Panel        
     Defined at drivers/video/mxc/Kconfig:55       
     Depends on: HAS_IOMEM [=y] && ARCH_MXC [=y] && FB_MXC_SYNC_PANEL [=y]
     Location:   
       -> Device Drivers    
         -> Graphics support 
           -> MXC Framebuffer support (FB_MXC [=y])    
             -> Synchronous Panel Framebuffer (FB_MXC_SYNC_PANEL [=y])   
```

更改LCD参数。			 

drivers/video/mxc/mxc_lcdif.c

```c
static struct fb_videomode lcdif_modedb[] = { 
	...
	{   
    /* 640x480 @ 60 Hz , pixel clk @ 25MHz */
    "SEIKO-WVGA", 60, 640, 480, 40000, 89, 164, 23, 10, 10, 10,
    FB_SYNC_CLK_LAT_FALL,
    FB_VMODE_NONINTERLACED,
    0,},
	...
};
```

Tony Liu 

2016-11-28, Shenzhen