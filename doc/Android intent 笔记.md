学习android的intent,将其中的一些总结，整理的笔记记录于此。

intent是一个消息传递对象，可以在不同组件间传递数据。Activity,Service,Broadcast Receiver中都会用到。

### 参考链接

---

https://developer.android.com/guide/components/intents-filters.html?hl=zh-cn#Receiving

https://www.tutorialspoint.com/android/android_intents_filters.htm

http://www.cnblogs.com/skynet/archive/2010/07/20/1781644.html

http://www.cnblogs.com/feisky/archive/2010/01/16/1649081.html


### intent主要信息

---

在介绍intent之前，需要了解intent中包含的信息。主要如下：

* componentName

要启动的目标组件名称。例如com.example.ExampleActivity。可以使用setComponent()，

* action

表示要执行的动作

* data

执行动作要操作的数据，android采用指向数据的URI和MIME类型来表示。

例如：

ACTION_MAIN 运行一个任务的第一个启动的activity

ACTION_VIEW content://contracts/people/1 ,显示ID是1的people信息。contentprovider使用

ACTION_DAIL tel:123 , 拨打电话号码123

* category

表示要处理intent的组件类型信息。这里指的是接收intent的对象的类型信息。

常见如下：

CATEGORY_LAUNCHER  表示activity是任务出事的activity。第一个启动的activity。

CATEGORY_BROWSABLE  目标Activity允许本省通过网络浏览器启动。

* extra

额外的信息，以键值对的形式保存。可以保存数据到intent中，在接收的地方读取出来。

* flag

可以指示android如何启动activity,以及启动后如何处理。

例如：

FLAG_ACTIVITY_CLEAR_TASK,在activity启动前清楚与它相关的activity,之后再启动。


### Intent的类型

---

Intent有两种类型，显示意图explicit intent和隐式意图implicit intent。下面会分别介绍。

* explicit intents

显式的intents，在使用时，明确指定激活的组件名称。例如：

```
	Intent intent = new Intent(this,Activity2.class);
	startActivity(intent);
```

启动名称是Activity2的组件。

* implicit intents

隐式intents。没有明确指定组件名称。根据intent中的action,category,数据(URI/MIME)等，找到最合适的组件。


### intent-filter

---

活动，服务，广播接收者为了告知系统能够处理那些隐式的intent，就将要接受的隐式intent信息写在intent-filter标签中。

app发出的intent之后，android系统就会根据intent-filter中的action,category等信息。只有匹配成功之后，接受到的intent才会被处理。


### example

---

android中创建工程，名称是"IntentTest"。

* 布局文件

设置三个按键，分别用于explicit intent,implicit intent和intent-filter测试。

activity_main.xml

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
    tools:context="com.example.intenttest.MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <Button
            android:id="@+id/explicit"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="explicit test"/>

        <Button
            android:id="@+id/implicit"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="implicit test"/>

        <Button
            android:id="@+id/intentFilter"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="intent filter test" />

    </LinearLayout>

</RelativeLayout>
```

* 新建activity

在Project窗口中，右键点击 app 文件夹并选择 New > Activity > Empty Activity。

activity名称Activity2。

android studio会新建一个Activity.java的文件，并在AndroidManifest.xml中添加activity.

Activity2.java如下所示：

```
package com.example.intenttest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class Activity2 extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_2);
    }
}
```

* AndroidManifest.xml

清单文件如下所示

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.intenttest">

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

        <activity android:name=".Activity2">
		</activity>
    </application>

</manifest>
```

现在的清单文件中已经有intent-filter标签。

ACTION_MAIN 操作指示这是主要入口点，且不要求输入任何 Intent 数据。

CATEGORY_LAUNCHER 类别指示此 Activity 的图标应放入系统的应用启动器。

所以一开始运行app时,第一个启动的是MainActivity，而不是新建的Activity2。

* MainActivity

MainActivity.java如下所示：

```
package com.example.intenttest;

import android.content.Intent;
import android.net.Uri;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {
    public Button btnExplicit;
    public Button btnImplicit;
    public Button btnIntentFilter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        btnExplicit = (Button) findViewById(R.id.explicit);
        btnImplicit = (Button) findViewById(R.id.implicit);
        btnIntentFilter = (Button) findViewById(R.id.intentFilter);
        // 打开activity2
        btnExplicit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 根据指定类型开启组件
                Intent intent = new Intent(MainActivity.this, Activity2.class);
                // 除了通过指定类型开启组件，还可以根据组件包名、全路径来指定开启组件。
                /*
                Intent intent = new Intent();
                intent.setClassName("com.example.intenttest","com.example.intenttest.Activity2");
                */
                startActivity(intent);
            }
        });
        // 打开浏览器，访问网址
        btnImplicit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                intent.setAction(android.content.Intent.ACTION_VIEW);
                intent.setData(Uri.parse("http://www.google.com"));
                startActivity(intent);
            }
        });
        // 打开相机
        btnIntentFilter.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                intent.setAction("android.media.action.IMAGE_CAPTURE");
                intent.addCategory("android.intent.category.DEFAULT");
                startActivity(intent);
            }
        });
    }
}
```

将app安装到手机上。

点击第一个按键“EXPLICIT TEST”将会切换到Activity2的界面，是一个空白的界面。

点击第一个按键“IMPLICIT TEST”将会打开手机的浏览器，并访问"www.google.com"，但由于goole被墙，访问失败。

点击第三个按键“INTENT FILTER TEST”将会直接带卡手机的相机。

ui如下：

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170308194246469-404436414.png)


* intent-filter测试

下面做一些修改，用于测试intent-filter。

在AndroidMainfest.xml中为Activity2添加<intent-filter>标签。

android.media.action.IMAGE_CAPTURE和打开相机的action是相同的。

如下为新的AndroidManifest.xml文件。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.intenttest">

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

        <activity android:name=".Activity2">
            <intent-filter>
                <action android:name="android.media.action.IMAGE_CAPTURE"/>
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

从新运行更改之后的app。当点击第三个按键“INTENT FILTER TEST”时，会弹出选框。选择相机还是IntentTest。

点击IntentTest,会直接跳转到Activity2的界面。说明设置的intent-filter生效了。如下所示：

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170308194313953-474480037.png)



Tony Liu

2017-3-8, Shenzhen