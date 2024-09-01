LayoutInflater类用于查找布局文件并实例化。用于动态将布局加入界面中。

### 参考链接

http://blog.csdn.net/guolin_blog/article/details/12921889

http://blog.sina.com.cn/s/blog_5da93c8f0100xm6n.html

### 分析

- 生成LayoutInflater实例2种方法

`LayoutInflater layoutInflater = LayoutInflater.from(context);`

或者

`LayoutInflater layoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);`

其中context是Activity的上下文内容。如果在activity中可以通过this表示，或者可通过getApplicationContext()获取。

- setContentview()区别

使用setContentView()函数设置了布局，会马上显示在UI中。而LayoutInflater的实例却不会。

- findViewById()

findViewById()只能查找setContentView()中设置的layout布局中的控件。

如果setContentView()所设置的布局中没有所要访问的控件，而在其他布局文件中，直接使用findViewById()访问会出错。

需要先通过LayoutInflater类现将对应的布局实例化之后,并添加到布局中(不添加也看不到控件).再通过findViewById()。

### Example

根据参考链接的源码，记录于此。

* activity_main.xml

主要的布局文件，一个RelativeLayout,里面什么也没有。

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
    android:id="@+id/main_layout"
    tools:context="com.example.inflatetest.MainActivity">

</RelativeLayout>
```

* button_layout.xml

将要将这个布局动态添加到activity_main.xml中。里面有LinearLayout和Button。

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        android:id="@+id/btn"
        android:text="button"
        />

</LinearLayout>
```

* AndroidManifest.xml

清单文件

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.inflatetest">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```


* ManiActivity.java

主要代码如下。
使用首先实例化LayoutInflater，

然后使用inflate()加载要添加的布局文件

最后使用addView()添加到activity_main.xml中。

之后使用findViewById()查找button才会成功。

```
package com.example.inflatetest;

import android.content.Context;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.RelativeLayout;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    private RelativeLayout mainLayout;

    Button btn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 创建LayoutInflater实例
        LayoutInflater layoutInflater = LayoutInflater.from(this);
        // 另外一种方法生成LayoutInflater实例。
//        LayoutInflater layoutInflater = (LayoutInflater) getApplicationContext().getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        // 通过LayoutInflater获取布局文件
        View buttonLayout = layoutInflater.inflate(R.layout.button_layout, null);

        // 将获取的布局添加到当前布局中。
        mainLayout = (RelativeLayout) findViewById(R.id.main_layout);
        mainLayout.addView(buttonLayout);

        // 使用inflater获取布局并添加之后,下面两种方法都可以。
//        btn = (Button) buttonLayout.findViewById(R.id.btn);
        btn = (Button) findViewById(R.id.btn);

        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(getApplicationContext(), "button click", Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```
显示效果：

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170317165948698-1562560024.png)


Tony Liu

2017-3-17, Shenzhen