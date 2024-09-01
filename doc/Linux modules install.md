安装模块的时候出现错误:modprobe: chdir(3.0.35-g6774ed9-dirty): No such file or directory.

内核模块没有安装正确。本文记录解决方法。

#### 参考链接

http://linfengdu.blog.163.com/blog/static/1177107320132710370696/

#### 问题描述

安装模块的时候出现错误。

```
root@freescale $ modprobe g_mass_storage.ko
modprobe: chdir(3.0.35-g6774ed9-dirty): No such file or directory
```

查看内核版本`uname -r`

```
root@freescale $ uname -r
3.0.35-g6774ed9-dirty
```

查看/lib/modules目录的模块安装位置：
```
root@freescale /lib/modules$ ls
3.0.35-2666-gbdde708
```

对应的目录名称与内核版本不对应。那么这个目录改怎么生成呢？

#### 模块编译

在内核源码根目录进行，模块编译,可以选择编译所有模块或者编译指定目录中的模块。

* 编译所有模块

`make modules`

* 编译指定目录中的模块

`make M=drivers/usb/gadget/ modules`

#### 安装模块

将模块安装到指定的路径。

* 默认安装的路径

`make modules_install`

默认安装在/lib/modules/kernel-version/

kernel-version是所编译的内核的版本

例如我编译的内核源码版本是`3.0.35-2666-gbdde708`

* 指定安装路径

`make modules_install INSTALL_MOD_PATH=~/rootfs`

指定到开发板的文件系统中或者打包放到文件系统中。

#### 查看安装结果

指定安装目录之后生成的结果

查看生成的内容如下

```
Qt@Tony:cd /home/Qt/rootfs/lib/modules/3.0.35-g6774ed9-dirty
Qt@Tony:~/rootfs/lib/modules/3.0.35-g6774ed9-dirty$ ll
total 148
drwxrwxr-x 3 Qt Qt  4096 Dec 14 13:09 ./
drwxrwxr-x 3 Qt Qt  4096 Dec 14 13:09 ../
lrwxrwxrwx 1 Qt Qt    20 Dec 14 13:08 build -> /home/Qt/kernel/
drwxrwxr-x 6 Qt Qt  4096 Dec 14 13:08 kernel/
-rw-rw-r-- 1 Qt Qt  8582 Dec 14 13:08 modules.alias
-rw-rw-r-- 1 Qt Qt  8513 Dec 14 13:08 modules.alias.bin
-rw-rw-r-- 1 Qt Qt 12237 Dec 14 13:08 modules.builtin
-rw-rw-r-- 1 Qt Qt 15275 Dec 14 13:08 modules.builtin.bin
-rw-rw-r-- 1 Qt Qt    69 Dec 14 13:08 modules.ccwmap
-rw-rw-r-- 1 Qt Qt  2812 Dec 14 13:08 modules.dep
-rw-rw-r-- 1 Qt Qt  5438 Dec 14 13:08 modules.dep.bin
-rw-rw-r-- 1 Qt Qt    75 Dec 14 13:08 modules.devname
-rw-rw-r-- 1 Qt Qt    73 Dec 14 13:08 modules.ieee1394map
-rw-rw-r-- 1 Qt Qt   141 Dec 14 13:08 modules.inputmap
-rw-rw-r-- 1 Qt Qt    81 Dec 14 13:08 modules.isapnpmap
-rw-rw-r-- 1 Qt Qt    74 Dec 14 13:08 modules.ofmap
-rw-rw-r-- 1 Qt Qt  2139 Dec 14 13:08 modules.order
-rw-rw-r-- 1 Qt Qt   463 Dec 14 13:08 modules.pcimap
-rw-rw-r-- 1 Qt Qt    43 Dec 14 13:08 modules.seriomap
-rw-rw-r-- 1 Qt Qt   131 Dec 14 13:08 modules.softdep
-rw-rw-r-- 1 Qt Qt  9248 Dec 14 13:08 modules.symbols
-rw-rw-r-- 1 Qt Qt 11607 Dec 14 13:08 modules.symbols.bin
-rw-rw-r-- 1 Qt Qt  6886 Dec 14 13:08 modules.usbmap
lrwxrwxrwx 1 Qt Qt    20 Dec 14 13:08 source -> /home/Qt/kernel/
```

modules.dep 用于记录模块的依赖关系。

如果没有可以使用depmod命令生成(这一块没有验证)。

模块位于kernel目录

```
Qt@Tony:~/kernel/module_install/lib/modules/3.0.35-g6774ed9-dirty$ tree kernel/
kernel/
├── crypto
│   └── tcrypt.ko
├── drivers
│   ├── gpu
│   │   └── drm
│   │       ├── drm.ko
│   │       └── vivante
│   │           └── vivante.ko
│   ├── hid
│   │   ├── hid-a4tech.ko
│   │   ├── hid-apple.ko
│   │   ├── hid-belkin.ko
│   │   ├── hid-cherry.ko
│   │   ├── hid-chicony.ko
│   │   ├── hid-cypress.ko
│   │   ├── hid-ezkey.ko
│   │   ├── hid-gyration.ko
│   │   ├── hid-logitech.ko
│   │   ├── hid-microsoft.ko
│   │   ├── hid-monterey.ko
│   │   ├── hid-petalynx.ko
│   │   ├── hid-pl.ko
│   │   ├── hid-samsung.ko
│   │   ├── hid-sony.ko
│   │   └── hid-sunplus.ko
│   ├── i2c
│   │   └── algos
│   │       └── i2c-algo-bit.ko
│   ├── media
│   │   └── video
│   │       ├── gspca
│   │       │   └── gspca_main.ko
│   │       ├── mxc
│   │       │   └── capture
│   │       │       ├── adv7180_tvin.ko
│   │       │       ├── camera_sensor_clock.ko
│   │       │       ├── ipu_bg_overlay_sdc.ko
│   │       │       ├── ipu_csi_enc.ko
│   │       │       ├── ipu_fg_overlay_sdc.ko
│   │       │       ├── ipu_prp_enc.ko
│   │       │       ├── ipu_still.ko
│   │       │       ├── mxc_v4l2_capture.ko
│   │       │       ├── ov3640_camera.ko
│   │       │       ├── ov5640_camera.ko
│   │       │       ├── ov5640_camera_mipi.ko
│   │       │       ├── ov5642_camera.ko
│   │       │       └── ov8820_camera_mipi.ko
│   │       └── uvc
│   │           └── uvcvideo.ko
│   ├── misc
│   │   └── mxs-perfmon.ko
│   ├── mxc
│   │   └── mlb
│   │       └── mxc_mlb150.ko
│   ├── net
│   │   └── wireless
│   │       ├── ath
│   │       │   └── ath6kl
│   │       │       └── ath6kl.ko
│   │       ├── hostap
│   │       │   └── hostap.ko
│   │       ├── rtl8192ce
│   │       │   └── 8192ce.ko
│   │       └── rtl8723bs
│   │           └── 8723bs.ko
│   ├── scsi
│   │   └── scsi_wait_scan.ko
│   └── usb
│       └── gadget
│           ├── g_file_storage.ko
│           └── g_mass_storage.ko
├── fs
│   ├── autofs4
│   │   └── autofs4.ko
│   ├── configfs
│   │   └── configfs.ko
│   └── nls
│       ├── nls_ascii.ko
│       └── nls_utf8.ko
└── net
    └── wireless
        ├── lib80211_crypt_ccmp.ko
        ├── lib80211_crypt_tkip.ko
        └── lib80211_crypt_wep.ko
33 directories, 51 files
```

Tony Liu

2016-12-14, Shenzhen