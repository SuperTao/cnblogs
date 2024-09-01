开发板需要使用spi接口，但是spi接口被touch占用，使用event进行操作。所以需要更改配置，生成spi设备节点。

##### 参考链接

　　https://community.nxp.com/thread/322097

　　http://blog.csdn.net/yaolanshu_june/article/details/52152790

##### 更改内核配置 

make menuconfig

添加spi的支持，如下所示。

```
Device Drivers
    [*] SPI support  --->
            --- SPI support                              
                   *** SPI Master Controller Drivers *** 
             < >   Altera SPI Controller                 
             -*-   Utilities for Bitbanging SPI masters  
             < >   GPIO-based bitbanging SPI Master      
             <*>   Freescale i.MX SPI controllers        
             < >   OpenCores tiny SPI                    
             < >   ARM AMBA PL022 SSP controller         
             < >   Xilinx SPI controller common module   
             < >   DesignWare SPI controller core support
                   *** SPI Protocol Masters ***          
             <*>   User mode SPI device driver support   
             < >   Infineon TLE62X0 (for power switching)
```

##### 添加spi设备

　　添加spi设备，名称一定要是'spidev'.

　　vi arch/arm/mach-mx6/board-mx6q_sabresd.c
```
static struct spi_board_info imx6_sabresd_spi_nor_device[] __initdata = {
    {    
        .modalias = "spidev",
        .max_speed_hz = 50000, /* max spi clock (SCK) speed in HZ */
        .bus_num = 1, 
        .chip_select = 0, 
        .mode = SPI_MODE_1,
    },   
};
```

从新编译内核，查看spi设备节点。
```
root@freescale ~$ ll /dev/spidev1.0 
crw-rw----    1 root     root      153,   0 Jan  1 00:00 /dev/spidev1.0
```

Tony Liu

2016-10-20, Shenzhen