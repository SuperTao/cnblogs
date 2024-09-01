手头上有一个320x240的LCD。运行android时，显示内容过大，需要更改屏幕的分辨率。

#### 参考链接

http://www.bkjia.com/Androidjc/899396.html

http://blog.csdn.net/AA747604141/article/details/18660329

http://blog.sina.com.cn/s/blog_67f5326b0102v8nz.html

http://www.jianshu.com/p/ec5a1a30694b

#### 更改方法

在/system/build.prop中添加

`ro.sf.lcd_density=120`

```
Android屏幕密度（Density）和分辨率的解释
HVGA屏density=160
QVGA屏density=120
WVGA屏density=240
WQVGA屏density=120
```

Tony Liu

2016-12-29, Shenzhen