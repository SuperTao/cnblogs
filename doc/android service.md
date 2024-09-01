学习android的service，了解bind的原理。将笔记记录于此。

参考连接：

http://blog.csdn.net/guolin_blog/article/details/11952435

http://www.cnblogs.com/codingblock/p/4842224.html

https://www.tutorialspoint.com/android/android_services.htm

service是在后台运行的服务，没有界面。首先看一张网上的图片。

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170302200953641-1168600834.jpg)


左边生命周期图是没有采用bindService()，只是调用了onStartCommand()。

右边是采用了bindService()的生命周期图。

清单文件如下所示

AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.tony.servicetest">

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
		
		<!-- 注册Myservice -->
        <service
            android:name=".MyService"
            android:enabled="true"
            android:exported="true"></service>
    </application>

</manifest>
```

ui的布局文件，如下所示：

activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <Button
        android:id="@+id/startService"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="start_service"/>

    <Button
        android:id="@+id/stopService"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="stop_service"/>

    <Button
        android:id="@+id/bindService"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="bind_service"/>

    <Button
        android:id="@+id/unbindService"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="unbind_service"/>
</LinearLayout>
```

ui的显示效果如下图所示。

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170302201326610-1021769089.png)


MainActivity


```
package com.example.tony.servicetest;

import android.content.ComponentName;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.IBinder;
import android.os.ParcelFileDescriptor;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private Button startServiceButton;
    private Button stopServiceButton;
    private Button bindServiceButton;
    private Button unbindServiceButton;

    private MyService.MyBinder myBinder;

    /* 创建匿名类
     * 在里面重写了onServiceConnected()方法和onServiceDisconnected()方法，
     */
    private ServiceConnection myConnection = new ServiceConnection() {
        /* bind成功之后，会运行onServiceConnected函数。
         * 这样就可以在activity中指挥service
         */
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            myBinder = (MyService.MyBinder) service;
            myBinder.startDownload();

            Log.i("MyActivity", "onServiceConnected() executed");
        }
        /*
         * 测试的时候，unbind时，并没有调用onServiceDisconnected函数，网上有解释如下，service异常断开连接的时候才会调用。
         * This is called when the connection with the service has been unexpectedly disconnected -- that is,
         * its process crashed. Because it is running in our same process, we should never see this happen.
         */
        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.i("MyActivity", "onServiceDisconnected() executed");
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        startServiceButton = (Button) findViewById(R.id.startService);
        stopServiceButton = (Button) findViewById(R.id.stopService);
        bindServiceButton = (Button) findViewById(R.id.bindService);
        unbindServiceButton = (Button) findViewById(R.id.unbindService);

        startServiceButton.setOnClickListener(this);
        stopServiceButton.setOnClickListener(this);
        bindServiceButton.setOnClickListener(this);
        unbindServiceButton.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
	        switch (v.getId()) {
            case R.id.startService:
                Log.i("MyActivity", "start_service button pressed");
                Intent startIntent = new Intent(this, MyService.class);
                startService(startIntent);      // 启动service
                break;
            case R.id.stopService:
                Log.i("MyActivity", "stop_service button pressed");
                Intent stopIntent = new Intent(this, MyService.class);
                stopService(stopIntent);        // 停止service
                break;
            // 将Activity和Service进行关联
            case R.id.bindService:
                Log.i("MyActivity", "bind_service button pressed");
                Intent bindIntent = new Intent(this, MyService.class);
                /* 这里传入BIND_AUTO_CREATE表示在Activity和Service建立关联后自动创建Service，
                 * 这会使得MyService中的onCreate()方法得到执行，但onStartCommand()方法不会执行。
                 */
                bindService(bindIntent, myConnection, BIND_AUTO_CREATE);
                break;
            // 将Activity和Service解除关联
            case R.id.unbindService:
                Log.i("MyActivity", "unbind_service button pressed");
                unbindService(myConnection);
                break;
            default:
                break;
        }
    }
}
```



MyService.java

```
package com.example.tony.servicetest;

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;
import android.util.Log;
import android.widget.Toast;

public class MyService extends Service {
    public static final String TAG = "MyService";

    private MyBinder mBinder = new MyBinder();
    // 服务首次创建的时候会运行onCreate
    @Override
    public void onCreate() {
        super.onCreate();
        Log.i(TAG, "onCreate() executed");
        Toast.makeText(this, "Service create", Toast.LENGTH_LONG).show();
    }
    // activity运行startService是调用。
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.i(TAG, "onStartcommand() executed");
        Toast.makeText(this, "Service Started", Toast.LENGTH_LONG).show();
        return super.onStartCommand(intent, flags, startId);
    }
	// activity运行bindService()时调用
    @Override
    public IBinder onBind(Intent intent) {
        Log.i(TAG, "onBind() executed");
        return mBinder;
    }
	
    @Override
    public boolean onUnbind(Intent intent){
        Log.i(TAG, "onUnbind() executed");
        /*
         * 这里需要return true,activity再次进行bind的时候才会调用onRebind函数。
         * return false不能调用，return super.onUnbind(intent)我试了也不能调用onRebind函数。
         */
        return true;
    }

    @Override
    public void onRebind(Intent intent) {
        super.onRebind(intent);
        Log.i(TAG, "onRebind() executed");
    }

    // activity运行stopService()时调用
    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.i(TAG, "onDestroy() executed");
        Toast.makeText(this, "Service Destroy", Toast.LENGTH_LONG).show();
    }

    class MyBinder extends Binder {
        /* 自己实现的一个函数，可以在activity中指定运行这个类中的函数。
		 * 绑定成功的话，指定的函数就会在service中运行。
		 */
        public void startDownload() {
            Log.i(TAG, "start download() executed");
        }
    }
}
```

测试的log信息如下。

先点击start_service button，再点击stop_service button。这样操作与最上面的图，没有bindService的时候吻合。

```
03-02 20:03:25.387 19557-19557/com.example.tony.servicetest I/MyActivity: start_service button pressed
03-02 20:03:25.393 19557-19557/com.example.tony.servicetest I/MyService: onCreate() executed
03-02 20:03:25.400 19557-19557/com.example.tony.servicetest I/MyService: onStartcommand() executed
03-02 20:03:28.817 19557-19557/com.example.tony.servicetest I/MyActivity: stop_service button pressed
03-02 20:03:28.820 19557-19557/com.example.tony.servicetest I/MyService: onDestroy() executed
```
下面的几个操作都是调用了bindService()。

start_service button-> bind_service button -> unbind_service button -> stop_service

```
03-02 20:04:13.573 19557-19557/com.example.tony.servicetest I/MyActivity: start_service button pressed
03-02 20:04:13.578 19557-19557/com.example.tony.servicetest I/MyService: onCreate() executed
03-02 20:04:13.581 19557-19557/com.example.tony.servicetest I/MyService: onStartcommand() executed
03-02 20:04:19.624 19557-19557/com.example.tony.servicetest I/MyActivity: bind_service button pressed
03-02 20:04:19.629 19557-19557/com.example.tony.servicetest I/MyService: onBind() executed
03-02 20:04:19.638 19557-19557/com.example.tony.servicetest I/MyService: start download() executed
03-02 20:04:19.638 19557-19557/com.example.tony.servicetest I/MyActivity: onServiceConnected() executed
03-02 20:04:25.180 19557-19557/com.example.tony.servicetest I/MyActivity: unbind_service button pressed
03-02 20:04:25.182 19557-19557/com.example.tony.servicetest I/MyService: onUnbind() executed
03-02 20:04:26.155 19557-19557/com.example.tony.servicetest I/MyActivity: stop_service button pressed
03-02 20:04:26.157 19557-19557/com.example.tony.servicetest I/MyService: onDestroy() executed
```

start_service button-> bind_service button -> unbind_service button -> bin_service button -> unbind_service -> stop service

这样操作，就调用了onRebind()函数。

```
03-02 20:04:55.657 19557-19557/com.example.tony.servicetest I/MyActivity: start_service button pressed
03-02 20:04:55.662 19557-19557/com.example.tony.servicetest I/MyService: onCreate() executed
03-02 20:04:55.666 19557-19557/com.example.tony.servicetest I/MyService: onStartcommand() executed
03-02 20:04:58.283 19557-19557/com.example.tony.servicetest I/MyActivity: bind_service button pressed
03-02 20:04:58.287 19557-19557/com.example.tony.servicetest I/MyService: onBind() executed
03-02 20:04:58.291 19557-19557/com.example.tony.servicetest I/MyService: start download() executed
03-02 20:04:58.292 19557-19557/com.example.tony.servicetest I/MyActivity: onServiceConnected() executed
03-02 20:04:59.246 19557-19557/com.example.tony.servicetest I/MyActivity: unbind_service button pressed
03-02 20:04:59.253 19557-19557/com.example.tony.servicetest I/MyService: onUnbind() executed
03-02 20:05:00.237 19557-19557/com.example.tony.servicetest I/MyActivity: bind_service button pressed
03-02 20:05:00.241 19557-19557/com.example.tony.servicetest I/MyService: start download() executed
03-02 20:05:00.241 19557-19557/com.example.tony.servicetest I/MyActivity: onServiceConnected() executed
03-02 20:05:00.241 19557-19557/com.example.tony.servicetest I/MyService: onRebind() executed
03-02 20:05:02.315 19557-19557/com.example.tony.servicetest I/MyActivity: unbind_service button pressed
03-02 20:05:02.317 19557-19557/com.example.tony.servicetest I/MyService: onUnbind() executed
03-02 20:05:03.023 19557-19557/com.example.tony.servicetest I/MyActivity: stop_service button pressed
03-02 20:05:03.026 19557-19557/com.example.tony.servicetest I/MyService: onDestroy() executed
```

Tony Liu

2017-3-2, Shenzhen