ActionBar是一种新増的导航栏功能，在Android 3.0之后加入到系统的API当中，它标识了用户当前操作界面的位置，并提供了额外的用户动作、界面导航等功能。

### 参考链接

https://developer.android.com/training/appbar/index.html?hl=zh-cn

http://blog.csdn.net/guolin_blog/article/details/18234477

在android studio中新建工程.默认的主题中就有action bar。也可以按照官网的操作，自己添加actionbar。

### 新建menu文件,用于设置actionbar的布局

res/menu/main_menu.xml

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    tools:context="com.example.actionbartest.MainActivity">

    <item
        android:id="@+id/pass"
        android:title="pass"
        android:icon="@drawable/test_pass"
        app:showAsAction="always" />

    <item
        android:id="@+id/fail"
        android:title="fail"
        android:icon="@drawable/test_fail"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/setting"
        android:title="setting"
        app:showAsAction="never" />

    <item
        android:id="@+id/configure"
        android:title="configure"
        app:showAsAction="never" />

</menu>
```

1.开头必须是menu标签。

2.item标签表示在menu中要显示的内容。

3.item的title标签在menu中的文字，在overflow中会显示。

4.item的icon标签是在menu显示的图片。这里使用了drawable目录中的png图片。

5.showAsAction中:

- always表示一直actionbar中显示出来,

- ifRoom表示如果空间足够，就在actionbar中显示，

- never表示在overflow(就是三个.的地方)中显示


### 布局文件

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
    tools:context="com.example.actionbartest.MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />
</RelativeLayout>
```

### 清单文件

AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.actionbartest">

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

默认的主题中就有actionbar.

### MainActivity.java

```
package com.example.actionbartest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuInflater;
import android.view.MenuItem;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.main_menu, menu);
        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.pass:
                Toast.makeText(this, "Pass", Toast.LENGTH_SHORT).show();
                return true;
            case R.id.fail:
                Toast.makeText(this, "fail", Toast.LENGTH_SHORT).show();
                return true;
            case R.id.setting:
                Toast.makeText(this, "setting", Toast.LENGTH_SHORT).show();
                return true;
            case R.id.configure:
                Toast.makeText(this, "configure", Toast.LENGTH_SHORT).show();
                return true;
            default:
                return super.onOptionsItemSelected(item);
        }
    }
}
```

1.重写onCreateOptionsMenu()函数，在app中就会显示actionbar添加的item.

2.重写onOptionsItemSelected()函数用于监听item的点击事件。

显示效果

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170316103048698-1988230258.png)



Tony Liu

2017-3-16, Shenzhen