最近需要更改im6 uboot的开机logo,使用10.1inch, 1024x600,18bit的LCD，期间遇到了很多的问题，记录于此。

#### 参考链接

　　https://community.nxp.com/message/650120?commentID=650120#comment-650120

　　https://community.nxp.com/thread/375767

　　https://sites.google.com/site/myembededlife/Home/u-boot/splashscreen

#### 1. 生成24bit的bmp图片

　　使用linux的gimp可以完成。gimp的使用可以参考我另一篇文章:[GIMP 使用](http://www.cnblogs.com/helloworldtoyou/p/5718486.html)

　　更改图片大小为1024x600,并另存为24bit的bmp图片。

　　经过尝试，一定要是24bit的bmp文件。

#### 2. 将图片转为.h文件

使用freescale的bin2txt工具进行转换。

- 下载地址

　　https://community.nxp.com/docs/DOC-93833

- 生成方法

　　python ./bin2txt tmp.bmp　　    //tmp.bmp是图片的名称

#### 3. 将生成的.h文件添加到uboot中

　　将生成的.h文件中数组中的内容替换board/freescale/common/fsl_bmp_reversed_600x400.c替换掉fsl_bmp_reversed_600x400数组中的内容。

#### 4. 更改uboot参数

　　board/freescale/mx6q_sabresd/mx6q_sabresd.c
```
//更改10.1寸屏幕参数。
static struct fb_videomode lvds_xgc = {
    "XGA", 60, 1024, 600, 19531, 150, 150, 15, 15, 20, 5,
    FB_SYNC_EXT,
    FB_VMODE_NONINTERLACED,
    0,
}

void lcd_enable(void)
{
    //18Bit
    ret = ipuv3_fb_init(&lvds_xga, di, IPU_PIX_FMT_RGB666, DI_PCLK_LDB, 65000000);  
    //24Bit
    //ret = ipuv3_fb_init(&lvds_xga, di, IPU_PIX_FMT_RGB24, DI_PCLK_LDB, 65000000); 
    ......

    if (di == 1)
    {
        //18bit
        writel(0x40C, IOMUXC_BASE_ADDR + 0x8);
        //24bit
        //writel(0x48C, IOMUXC_BASE_ADDR + 0x8);
    }
    else
    {
        //18bit
        writel(0x201, IOMUXC_BASE_ADDR + 0x8);//18BIT
        //24bit
        //writel(0x221, IOMUXC_BASE_ADDR + 0x8);//24BIT
    }
}
```

#### Author

Tony Liu

2016-8-10, Shenzhen