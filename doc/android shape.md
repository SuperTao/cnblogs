使用shape绘制图形，了解shape的使用。

　　　　　　　　　　　　　　　　　　　Tony,　2016-6-21, Shenzhen

参考链接

http://keeganlee.me/post/android/20150830

drawable目录添加xml文件,例如叫circle.xml

``` 
<?xml version="1.0" encoding="UTF-8"?>
<shape
xmlns:android="http://schemas.android.com/apk/res/android"
android:shape="oval"
android:useLevel="false" >
<padding
    android:bottom="4dp"
    android:left="4dp"
    android:right="4dp"
    android:top="4dp" />
<!-- 填充色 -->
<solid android:color="@android:color/holo_red_light" />
<!-- 线的宽度 -->
<stroke
    android:width="8dp"
    android:color="@android:color/holo_blue_dark" />
<!-- 圆宽度 -->
<size android:width="80dp"
    android:height="80dp" />
</shape>
```
layout/activity_main进行引用，例如使用textview

```
    <TextView
        android:id="@+id/ball"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="150dp"
        android:layout_marginTop="300dp"
        android:background="@drawable/circle"
        android:gravity="center"
        android:textSize="50dp"
        android:text="T"
        android:textColor="@android:color/white" />
```

显示效果
![](http://images2015.cnblogs.com/blog/745188/201606/745188-20160621230545803-1213704335.png)