菜单是应用中常见的用户组件。本文介绍如何在布局文件和代码中添加menu，submenu以及在代码中添加的方法。

### 参考链接

https://developer.android.com/guide/topics/ui/menus.html?hl=zh-cn

http://www.cnblogs.com/smyhvae/p/4133292.html

### 布局文件

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
    tools:context="com.example.menutest.MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />
</RelativeLayout>
```

### menu的布局文件

放置了一个start选项，以及一个set选项，并在set里面设置了2个子菜单set1和set2.

main_menu.xml

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/start"
        android:title="start"
        app:showAsAction="always" />

    <item
        android:id="@+id/set"
        android:title="set"
        app:showAsAction="always" >
        <menu>
            <item
                android:id="@+id/set1"
                android:title="set1"
                app:showAsAction="always" />

            <item
                android:id="@+id/set2"
                android:title="set2"
                app:showAsAction="always" />
        </menu>
        </item>

</menu>
```

### 清单文件

AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.menutest">

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

### MainActivity.java

重写onCreateOptionsMenu (Menu menu)函数，并调用inflate(R.menu.main_menu, menu)。这样menu的布局文件中的空间才会显示出来。

重写onOptionsItemSelecte()对菜单的点击事件进行监听。

addSubMenu()添加子菜单。并调用add()函数在子菜单中添加选项。

调用menu.add()在菜单中添加一个选项。 

```
package com.example.menutest;

import android.graphics.Bitmap;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuInflater;
import android.view.MenuItem;
import android.view.SubMenu;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    private static final int CONF_ITEM = Menu.FIRST;
    private static final int CONF_ITEM_1= Menu.FIRST + 1;
    private static final int CONF_ITEM_2 = Menu.FIRST + 2;
    private static final int STOP_ITEM = Menu.FIRST + 3;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    public boolean onCreateOptionsMenu (Menu menu) {
        int groupId = 0;

        /*
         * 添加子菜单,并在子菜单中添加选项
         * addSubMenu(final int groupId, final int itemId, int order, final CharSequence title);
         * groupId: 组ID,有时候会将菜单放在同一个组中，同时对组进行相关的设置。
         * itemId:  选项的id。点击选项的时候会根据这个来进行区分。
         * order:   顺序
         * title:   选项的标题
         */
        SubMenu sub1 = menu.addSubMenu(groupId, CONF_ITEM, Menu.NONE, "CONF");
        /*
         * 为子菜单中添加选项。
         * add(int groupId, int itemId, int order, CharSequence title);
         * groupId: 组ID,有时候会将菜单放在同一个组中，同时对组进行相关的设置。
         * itemId:  选项的id。点击选项的时候会根据这个来进行区分。
         * order:   顺序
         * title:   选项的标题
         */
        sub1.add(groupId, CONF_ITEM_1, Menu.NONE, "CONF_1");
        sub1.add(groupId, CONF_ITEM_2, Menu.NONE, "CONF_2");

        // 添加一个菜单选项
        menu.add(groupId + 1, STOP_ITEM, Menu.NONE, "STOP");

        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.main_menu, menu);

        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.start:
                Toast.makeText(getBaseContext(), "start", Toast.LENGTH_SHORT).show();
                break;

            case R.id.set:
                Toast.makeText(getBaseContext(), "setting", Toast.LENGTH_SHORT).show();
                break;

            case R.id.set1:
                Toast.makeText(getBaseContext(), "set 1", Toast.LENGTH_SHORT).show();
                break;

            case R.id.set2:
                Toast.makeText(getBaseContext(), "set 2", Toast.LENGTH_SHORT).show();
                break;

            case STOP_ITEM:
                Toast.makeText(getBaseContext(), "stop", Toast.LENGTH_SHORT).show();
                break;

            case CONF_ITEM:
                Toast.makeText(getBaseContext(), "conf", Toast.LENGTH_SHORT).show();
                break;

            case CONF_ITEM_1:
                Toast.makeText(getBaseContext(), "conf 1", Toast.LENGTH_SHORT).show();
                break;

            case CONF_ITEM_2:
                Toast.makeText(getBaseContext(), "conf 2", Toast.LENGTH_SHORT).show();
                break;

            default:
                break;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

Tony Liu

2017-3-16, Shenzhen