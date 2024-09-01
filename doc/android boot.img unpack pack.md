每次编译boot.img都要花比较长的时间，有时候只是更改其中的配置文件。

如果能够将boot.img解压，更改之后再打包的话，就能节省时间。

boot.img tools是别人写好的工具，能很好的解决boot.img解包的问题。

#### 参考链接：

http://mtksupport.blogspot.ru/2015/07/tool-bootimg-tools-unpack-repack-ramdisk.html

https://forum.xda-developers.com/showthread.php?t=2319018


#### boot.img unpack

从上面的网址中下载boot.img tool。我下载的是bootimg_tools_7.8.13.zip。

解压之后会有如下几个文件。

```
tony@tony:~/myandroid/out/target/product/sabresd_6dq/bootimg_tools$ ll -h
total 4.9M
drwxrwxr-x  3 tony tony 4.0K Feb 24 17:39 ./
drwxrwxr-x 11 tony tony 4.0K Feb 24 11:03 ../
-rwxrwxr-x  1 tony tony 5.0K Feb 24 11:03 boot_info*		# 查看boot.img信息 
-rwxrwxr-x  1 tony tony  28K Feb 24 11:03 mkbootimg*		# 打包boot.img
-rwxrwxr-x  1 tony tony  371 Feb 24 11:03 repack_ramdisk*	# 打包ramdisk
-rwxrwxr-x  1 tony tony 7.2K Feb 24 11:03 split_boot*		# 分离bootimg
-rwxrwxr-x  1 tony tony 580899 Feb 27 13:24 umkbootimg*
-rwxrwxr-x  1 tony tony    257 Feb 27 13:24 unpack*
-rwxrwxr-x  1 tony tony  287 Feb 24 11:03 unpack_ramdisk*	# 解压ramdisk
```

boot_info查看boot.img的信息，后续从新打包的时候会用到。

```
tony@tony:~/myandroid/out/target/product/sabresd_6dq/bootimg_tools$ ./boot_info ../boot.img 
PAGE SIZE: 2048
BASE ADDRESS: 0x10800000
RAMDISK ADDRESS: 0x11800000
CMDLINE: 'console=ttymxc0,115200 init=/init video=mxcfb0:dev=ldb,bpp=32 video=mxcfb1:off video=mxcfb2:off fbmem=10M fb0base=0x27b00000 vmalloc=400M androidboot.console=ttymxc0 androidboot.hardware=freescale'
```

* boot.img 解包

```
tony@tony:~/myandroid/out/target/product/sabresd_6dq/bootimg_tools$ ./split_boot ../boot.img 
Page size: 2048 (0x00000800)
Kernel size: 4825724 (0x0049a27c)
Ramdisk size: 183962 (0x0002ce9a)
Second size: 0 (0x00000000)
Board name: 
Command line: 'console=ttymxc0,115200 init=/init video=mxcfb0:dev=ldb,bpp=32 video=mxcfb1:off video=mxcfb2:off fbmem=10M fb0base=0x27b00000 vmalloc=400M androidboot.console=ttymxc0 androidboot.hardware=freescale'
Base address: (0x10800000)

Writing boot/boot.img-kernel ... complete.
Writing boot/boot.img-ramdisk.cpio.gz ... complete.
Unpacking ramdisk... complete.
```

在当前目录生成了一个boot的目录,内容如下:

```
tony@tony:~/myandroid/out/target/product/sabresd_6dq/bootimg_tools$ ls boot -lh
total 4.8M
-rw-rw-r-- 1 tony tony 4.7M Feb 27 11:26 boot.img-kernel
-rw-rw-r-- 1 tony tony 180K Feb 27 11:26 boot.img-ramdisk.cpio.gz
drwxrwxr-x 8 tony tony 4.0K Feb 24 13:26 ramdisk
```

boot.img-kernel是kenel的镜像

boot.img-ramdisk.cpio.gz是ramdisk的压缩包。

ramdisk是boot.img-ramdisk.cpio.gz解压之后的目录。

#### boot.img pack

* ramdisk pack

从新打包ramdisk。

```
tony@tony:~/myandroid/out/target/product/sabresd_6dq/bootimg_tools$ ./repack_ramdisk boot/ramdisk/
625 blocks

#在 boot目录生成new-ramdisk.cpio.gz文件
tony@tony:~/myandroid/out/target/product/sabresd_6dq/bootimg_tools$ ls boot
boot.img-kernel  boot.img-ramdisk.cpio.gz  new-ramdisk.cpio.gz  ramdisk
```

* boot.img pack

从新打包boot.img，命令行比较长，所以写成一个脚本。

repack_boot.sh

```
#!/bin/sh

set -x  # 脚本运行时，在终端输出每条运行的命令。

cmdline="console=ttymxc0,115200 init=/init video=mxcfb0:dev=ldb,bpp=32 video=mxcfb1:off video=mxcfb2:off fbmem=10M fb0base=0x27b00000 vmalloc=400M androidboot.console=ttymxc0 androidboot.hardware=freescale"

page_size=2048

base_addr=0x10800000

ramdisk_addr=0x11800000

./repack_ramdisk boot/ramdisk

./mkbootimg --cmdline "$cmdline" --base $base_addr --ramdiskaddr $ramdisk_addr --pagesize $page_size --kernel boot/boot.img-kernel --ramdisk boot/new-ramdisk.cpio.gz -o boot.img
```

运行时，输出如下所示。在当前目录生成boot.img文件。

```
tony@tony:~/myandroid/out/target/product/sabresd_6dq/bootimg_tools$ ./repack_boot.sh 
+ cmdline=console=ttymxc0,115200 init=/init video=mxcfb0:dev=ldb,bpp=32 video=mxcfb1:off video=mxcfb2:off fbmem=10M fb0base=0x27b00000 vmalloc=400M androidboot.console=ttymxc0 androidboot.hardware=freescale
+ page_size=2048
+ base_addr=0x10800000
+ ramdisk_addr=0x11800000
+ ./repack_ramdisk boot/ramdisk
625 blocks
+ ./mkbootimg --cmdline console=ttymxc0,115200 init=/init video=mxcfb0:dev=ldb,bpp=32 video=mxcfb1:off video=mxcfb2:off fbmem=10M fb0base=0x27b00000 vmalloc=400M androidboot.console=ttymxc0 androidboot.hardware=freescale --base 0x10800000 --ramdiskaddr 0x11800000 --pagesize 2048 --kernel boot/boot.img-kernel --ramdisk boot/new-ramdisk.cpio.gz -o boot.img
```

Tony Liu

2017-2-27, Shenzhen