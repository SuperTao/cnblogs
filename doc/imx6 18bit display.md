imx6 kernel中使用18bit的lcd，uboot中bootargs参数bpp=32，lcd才能够正常显示。

```
"bootargs=console=ttymxc0,115200 init=/init video=mxcfb0:dev=ldb,bpp=32 video=mxcfb1:off video=mxcfb2:off fbmem=40M vmalloc=400M androidboot.console=ttymxc0 androidboot.hardware=freescale\0" \
```

对应的kernel/arch/arm/mach-mx6/board-mx6q_sabresd.c更改如下：

```
static struct ipuv3_fb_platform_data sabresd_fb_data[] = {
    { /*fb0*/
    .disp_dev = "ldb",
    // 根据需要更改为RGB666,RGB24,GRB24或者其他
    .interface_pix_fmt = IPU_PIX_FMT_RGB666,     // 18bit
    //.interface_pix_fmt = IPU_PIX_FMT_RGB24,    // 24bit, RGB
    //.interface_pix_fmt = IPU_PIX_FMT_BGR24,    // 24bit, GRB
    .mode_str = "LDB-XGA",
    .default_bpp = 16,
    //.default_bpp = 24,
    .int_clk = false,
    .late_init = false,
    }, { 
    .disp_dev = "hdmi",
    .interface_pix_fmt = IPU_PIX_FMT_RGB24,
    .mode_str = "1920x1080M@60",
    .default_bpp = 32,
    .int_clk = false,
    .late_init = false,
    }, 
    ... 


```

网上也有类似的情况。

　　http://blog.csdn.net/xnwyd/article/details/11671123

为何bpp=32设置时，显示正确，还没有跟踪代码。

##### Author

Tony Liu

2016-8-9, Shenzhen