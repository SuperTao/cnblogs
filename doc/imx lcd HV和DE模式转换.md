有些时候拿到的lcd手册中关于芯片的时序使用的DE模式的，而imx6内核中使用的参数设置趋势HV模式，应此就需要将DE模式的参数转化为HV模式。

参考链接：

　　https://community.nxp.com/docs/DOC-93617
    
　　http://blog.csdn.net/liuhuahan/article/details/44172719

HV模式内核中数据结构

```
struct fb_videomode {
     const char*name;     /* optional */
     u32refresh;          /* optional */
     u32 xres;            /* 屏幕宽度 */
     u32 yres;            /* 屏幕高度 */
     u32 pixclock;        /* 屏幕时钟周期，单位是皮秒，datasheet是MHZ，需要转换 */
     u32 left_margin;     /* 左边间距 */
     u32 right_margin;    
     u32 upper_margin;    /* 上边距 */
     u32 lower_margin;
     u32 hsync_len;
     u32 vsync_len;
     u32 sync;
     u32 vmode;
     u32 flag;
};
```