imx6运行qt，在24bit的LVDS接口屏上显示时，显示效果与实际的不同。蓝色变成了黄色。

本来应该显示成蓝色：

![](http://images2015.cnblogs.com/blog/745188/201702/745188-20170218161445847-1504398343.png)

实际上去显示成了黄色：

![](http://images2015.cnblogs.com/blog/745188/201702/745188-20170218161459504-2002357261.png)

而其他绿色的图标并没有改变，只是蓝色和黄色互换了。

猜想应该是显示的时候RGB顺序的数据，当成了BGR来显示才会出现这种情况。

更改板级文件board-mx6q_sabresd.c的LVDS显示参数，由RGB改为BGR，如下    

```
static struct ipuv3_fb_platform_data sabresd_fb_data[] = {
    { /*fb0*/
    .disp_dev = "ldb",
	//.interface_pix_fmt = IPU_PIX_FMT_RGB24,
	.interface_pix_fmt = IPU_PIX_FMT_BGR24,
```

结果也只是boot logo的企鹅颜色变了，而QT的显示界面并没有变化。

上网查阅，直到找到下面的连接:

https://forum.qt.io/topic/4154/solved-setting-qt-for-embedded-pixel-type-to-24bpp-bgr/3

根据链接中的描述，更改qt的源码，从新编译之后，问题解决。

更改文件qt-everywhere-opensource-src-4.8.5/src/gui/embedded/qscreen_qws.cpp

```
void qt_blit_setup(QScreen *screen, const QImage &image,
                   const QPoint &topLeft, const QRegion &region)
{
    switch (screen->depth()) {
#ifdef QT_QWS_DEPTH_32
    case 32:
        if (screen->pixelType() == QScreen::NormalPixel)
            screen->d_ptr->blit = blit_32;
        else 
            screen->d_ptr->blit = blit_template<qabgr8888, quint32>;
        break;
#endif
// 更改24bit的参数
#ifdef QT_QWS_DEPTH_24
    case 24:
        if (screen->pixelType() == QScreen::NormalPixel)
            // Tony , 2017-2-18
            //screen->d_ptr->blit = blit_qrgb888;
            screen->d_ptr->blit = blit_24;
        else 
            screen->d_ptr->blit = blit_template<qrgb888, quint24>;
            //screen->d_ptr->blit = blit_24;
        break;
#endif
```

Tony Liu

2017-2-18, Shenzhen