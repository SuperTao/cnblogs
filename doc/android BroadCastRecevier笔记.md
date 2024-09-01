学习android的Broadcast，笔记记录于此。

BroadCastRecevier用于接受其他应用发送的广播.

BroadCastReceiver工作，需要2步。

* 创建Broadcast Recevier.

* 注册Braodcast Recevier.

#### 参考链接

https://www.tutorialspoint.com/android/android_broadcast_receivers.htm

http://blog.csdn.net/mr_dsw/article/details/51394350

#### 创建BroadcastReceiver

Android stuido创建project, 名称broadcastTest。

继承BroadcastReceiver，用于接收广播。

MyReceiver.java

```
package com.example.tony.broadcasttest;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.util.Log;
import android.widget.Toast;

public class MyReceiver extends BroadcastReceiver {
	// 接受到广播调用
    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
//        throw new UnsupportedOperationException("Not yet implemented");
        Log.i("receive", "onReceive");
        Toast.makeText(context, "intent Detected", Toast.LENGTH_LONG).show();
    }
}
```

#### 注册receiver

使用<receiver />标签注册broadcastReceiver。

AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.tony.broadcasttest">

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
		<!-- 注册Receiver，并定义action 
			 action name自己定义，但是发送广播的时候也要发送相同的广播，Receiver才能接收到 -->
        <receiver
            android:name=".MyReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="com.example.CUSTOM_INTENT" />
            </intent-filter>
        </receiver>

    </application>

</manifest>
```

#### 布局文件

布局文件中定义一个按键用于发送广播

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
    tools:context="com.example.tony.broadcasttest.MainActivity">

    <Button
        android:id="@+id/send_broadcast"
        android:text="send broadcast"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</RelativeLayout>
```

#### MainActivity

按键按下，通过sendBroadcast()发送自己定义的广播。

MainActivity.java

```
package com.example.tony.broadcasttest;

import android.content.Intent;
import android.content.IntentFilter;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {
    Button sendBroadcast;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        sendBroadcast = (Button) findViewById(R.id.send_broadcast);
		// 按键按下，发送广播
        sendBroadcast.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
               broadcastIntent(v);
            }
        });
        // 手动注册广播，在AndroidManifest.xml中注册了就不需要手动注册
		/*
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction("com.example.CUSTOM_INTENT");
        registerReceiver(new MyReceiver(), intentFilter);
        */
    }

    public void broadcastIntent(View view) {
        Intent intent = new Intent();
        // 发送自己广播，名称自己定义，只要在AndroidManifest.xmL中注册，让Receive接受即可。
        intent.setAction("com.example.CUSTOM_INTENT");
        sendBroadcast(intent);
        Log.i("sendBraodcast", "braodcastintent");
    }
}
```

这个demo只是无序广播的demo。对于有序广播，后面再分析。

Tony Liu

2017-3-3, Shenzhen