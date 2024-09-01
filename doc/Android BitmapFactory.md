android BitmapFactory

BitmapFactory是一个工具类，用于从不同数据源解析，创建Bitmap对象。bitmap类代表位图。

### BitmapFactory常用方法

**static Bitmap decodeFile(String pathName)**

从指定文件中解析,创建Bitmap对象。

**static Bitmap decodeStream(Inputstream is)** 

从指定输入流中解析、创建Bitmap对象。

**static Bitmap decodeResource(Resources res, int id)**

根据给定的资源id，解析创建Bitmap对象。

使用方法如下：

### Example

使用BitmapFactory.decodeResource()创建Bitmap对象。

再使用ImageView的setImageBitmap()在ImageView中显示。

**activity_main.xml**

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.example.imagetest.MainActivity">

    <ImageView
        android:id="@+id/imageViewPicture"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</RelativeLayout>
```

**MainActivity.java**

```
package com.example.imagetest;

import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.ImageView;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.pic);    //pic.png位于drawable目录

        ImageView imgView = (ImageView) findViewById(R.id.imageViewPicture);
        imgView.setImageBitmap(bitmap);
    }
}
```

Tony Liu

2017-3-17, Shenzhen