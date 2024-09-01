一个activity打开新的activity，新的activity关闭之后，返回数据。原来的activity要接收返回的数据，在开启新的activity时，就需要调用startActivityForResult()函数。

### 分析

* startActivityForResult()

原型如下：

startActivityForResult(Intent intent, int requestCode)

第一个参数Intent。

第二个参数requestCode: 请求码，返回的时候需要用到。可以自己设置。 

* onActivityResult()

如果要接受返回的数据。需要调用onActivityResult()

原型如下：

onActivityResult(int requestcode， int resultCode, Intent data)

第一个参数requestCode，与startActivityForResult对应。在onActivityResult函数中，根据不同的请求码进行操作。

第二个参数resultCode, 是setResult()函数返回的结果码。

第三个参数data，是数据。

* setResult()

对于打开的Activity，返回数据需要调用setResult()，将数据返回给前一个Activity。

原型如下：

setResult(int resultCode, Intent data) 

第一个参数resultCode是结果码，系统定义了静态变量，如RESULT_OK，RESULT_CANCELED等。

第二个参数data是返回的数据。

### Example

源码下载地址：https://github.com/TonySudo/StartActivityForResult

本例中，使用2个按键，调用startActivityForResult()，启动同一个新的activity，发送不同的请求码，数据也不同。

新的activity接受数据，并显示出来，然后将接受的数据通过setResult()返回。

原来的activity接受到数据，调用onActivityResult()，根据请求码和结果吗进行处理，将数据显示出来。

* AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.startactivityforresult">

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
        <activity android:name=".Activity2"></activity>
    </application>

</manifest>
```

* MainActivity的布局文件

设置2个按键，和2个textView用于显示。

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
    tools:context="com.example.startactivityforresult.MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <TextView
            android:id="@+id/textView1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!" />

        <TextView
            android:id="@+id/textView2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!" />

        <Button
            android:id="@+id/btnChangeText1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="change_text_1"/>

        <Button
            android:id="@+id/btnChangeText2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="change_text_2"/>


    </LinearLayout>

</RelativeLayout>
```

* MainActivity.java

按键按下，发送不同的requestCode和数据。

```
package com.example.startactivityforresult;

import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;


public class MainActivity extends AppCompatActivity {
    Button btn1;
    Button btn2;
    TextView tv1;
    TextView tv2;
    int requestCode = 0;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        btn1 = (Button) findViewById(R.id.btnChangeText1);
        btn2 = (Button) findViewById(R.id.btnChangeText2);

        btn1.setOnClickListener(listener);
        btn2.setOnClickListener(listener);

    }

    private View.OnClickListener listener = new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            switch (v.getId()) {
                case R.id.btnChangeText1:
                    requestCode = 1;
                    Intent intent1 = new Intent(MainActivity.this, Activity2.class);
                    intent1.putExtra("action", "Button1 pressed");
                    startActivityForResult(intent1, requestCode);
                    Log.e("test", "btn 1");
                    break;

                case R.id.btnChangeText2:
                    requestCode = 2;
                    Intent intent2 = new Intent(MainActivity.this, Activity2.class);
                    intent2.putExtra("action", "Button2 pressed");
                    startActivityForResult(intent2, requestCode);
                    Log.e("test", "btn 2");
                    break;

                default:
                    break;
            }

        }
    };
    /*
     * 获取上一个activity返回的数据。
     * requestcode: 调用startActivityForResult时，传入的参数的，返回的时候也会穿这个参数。
     * resultCode:  setResult()时传入的数据。表示操作是否成功。有多重参数，根据不同的记过进行操作。
     * data:        返回的数据。
     */
    @Override
    protected void onActivityResult(int requestcode, int resultCode, Intent data) {
        if (resultCode == RESULT_OK) {
            String str = data.getStringExtra("action");
            switch (requestCode) {
                case 1:
                    tv1 = (TextView) findViewById(R.id.textView1);
                    tv1.setText(str);
                    break;
                case 2:
                    tv2 = (TextView) findViewById(R.id.textView2);
                    tv2.setText(str);
                    break;
            }
        }
        else if (resultCode == RESULT_CANCELED) {

        }
    }
}
```

* Activity2的布局文件

activity_2.xml

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
    tools:context="com.example.startactivityforresult.Activity2">

    <TextView
        android:id="@+id/textViewActivity2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</RelativeLayout>
```

* Activity2.java

```
package com.example.startactivityforresult;

import android.content.ContentUris;
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.TextView;
import android.widget.Toast;

public class Activity2 extends AppCompatActivity {
    TextView txtV;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_2);

        // 接受上一个activity发过来的数据并显示。
        Intent intent = getIntent();
        String str = intent.getStringExtra("action");
        txtV = (TextView) findViewById(R.id.textViewActivity2);
        txtV.setText(str);

        // 发送接收到的数据，发还给上一个activity。
        Intent intentBack = new Intent();
        intentBack.putExtra("action", str);
        setResult(RESULT_OK, intentBack);
//        setResult(RESULT_CANCELED, intentBack);
        // 如果立即调用finish()，这个activity会马上关闭，返回到前一个activity。所以不再这里调用。放到onStop()中调用。
//        this.finish();
    }

    @Override
    public void onStop()
    {
        super.onStop();
        this.finish();
        Toast.makeText(getBaseContext(), "activity2 stop", Toast.LENGTH_SHORT).show();
    }
}
```

运行效果：

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170310150119139-814733431.png)

点击第一个按键

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170310150133951-342535479.png)

返回

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170310150142732-399363935.png)


再点击第二个按键

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170310150151576-1782000035.png)

再返回

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170310150159467-1948239322.png)


Tony Liu

2017-3-10，Shenzhen