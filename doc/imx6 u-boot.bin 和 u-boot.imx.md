有些MFG TOOL烧录工具使用了u-boot.imx，而不是原来的u-boot.bin文件进行烧录。

这两个镜像的区别是，u-boot.bin文件编译后，会在u-boot.bin的开头添加一个大小为1K的IVT头，用于告诉BOOT ROM找到uboot的位置和函数,要运行在什么模式，DRAM的配置数据等。新生成的文件就是u-boot.imx文件。

参考链接：

https://community.nxp.com/thread/309765

以下内容来自参考链接

```
You can look into the HEX code of u-boot.imx, for the first 1K structure, the beginning is formatted as below, this header is for uboot-v2009.08, but I think uboot-2013-04 is same, although the code is different, but eventually they will both put a header in front of uboot.bin.
 
ROM will read first IVT header to identify which mode need to execute, DCD or PLUG, if DCD mode, then where to find the DRAM config data, and after DRAM configured, where to read the reset uboot image and where to put this image, in which DRAM address etc.
```

Tony Liu

2016-11-11, Shenzhen