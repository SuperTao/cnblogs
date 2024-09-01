最近调试imx6的屏幕显示，笔记记录于此。

官方文档关于uboot参数的介绍：

![](http://images2015.cnblogs.com/blog/745188/201704/745188-20170427103617350-972491815.png)

sin和dul参数已经测试过，sep和spl还没有验证成功。

```
1 单屏显示
说明：输入命令并按确定键， 观察系统启动过程中显示屏的显示内容，即可看到 Linux
Logo。

1.1 LVDS1
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=ldb,LDB-XGA,if=RGB666

1.2 LVDS0
进入 u-boot 命令行，输入下面命令并按确定键：

setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=ldb,LDB-XGA,if=RGB666 ldb=sin0

1.3 HDMI
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=hdmi,1920x1080M@60,if=RGB24

1.4 RGB
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=lcd,SEIKO-WVGA,if=RGB24


2 双屏同步骤显示

2.1 LVDS1+LVDS0 同步显示
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=ldb,LDB-XGA,if=RGB666 ldb=dul0 video=mxcfb1:dev=ldb,LDB-XGA,if=RGB666

3. 双屏异步显示

3.1 LVDS1 作为主屏

3.1.1 LVDS1+LVDS0 双屏异步显示
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=ldb,LDB-XGA,if=RGB666 video=mxcfb1:dev=ldb,LDB-XGA,if=RGB666

3.1.2 LVDS1+RGB 双屏异步显示
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=ldb,LDB-XGA,if=RGB666 video=mxcfb1:dev=lcd, SEIKO-WVGA,if=RGB24

3.1.3 LVDS1+HDMI 双屏异步显示
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=ldb,LDB-XGA,if=RGB666 video=mxcfb1:dev=hdmi,1920x1080M@60,if=RGB24

3.2 LVDS0 作为主屏
3.2.1 LVDS0+LVDS1 双屏异步显示：
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=ldb,LDB-XGA,if=RGB666 ldb=sep0 video=mxcfb1:dev=ldb,LDB-XGA,if=RGB666

3.2.2 LVDS0+RGB 双屏异步显示
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=ldb,LDB-XGA,if=RGB666 ldb=sin0 video=mxcfb1:dev=lcd,SEIKO-WVGA,if=RGB24

3.2.3 LVDS0+HDMI 双屏异步显示
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=ldb,LDB-XGA,if=RGB666 ldb=sin0 video=mxcfb1:dev=hdmi,1920x1080M@60,if=RGB24

3.3 RGB 作为主屏
3.3.1 RGB+LVDS1 双屏异步显示：
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=lcd,SEIKO-WVGA,if=RGB24 video=mxcfb1:dev=ldb,LDB-XGA,if=RGB666

3.3.2 RGB+LVDS0 双屏异步显示：
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=lcd,SEIKO-WVGA,if=RGB24 video=mxcfb1:dev=ldb,LDB-XGA,if=RGB666 ldb=sin0

3.4 HDMI 作为主屏
3.4.1 HDMI+LVDS1 双屏异步显示
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=hdmi,1920x1080M@60,if=RGB24 video=mxcfb1:dev=ldb,LDB-XGA,if=RGB666

3.4 2 HDMI+LVDS0 双屏异步显示
setenv bootargs console=ttymxc0,115200 ip=none root=/dev/mmcblk0p1 rootwait video=mxcfb0:dev=hdmi,1920x1080M@60,if=RGB24 video=mxcfb1:dev=ldb,LDB-XGA,if=RGB666 ldb=sin0
tm
```

Tony Liu

2017-4-27, Shenzhen