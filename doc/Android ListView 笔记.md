ListView以列表的形式展示数据内容。

布局文件如下所示,添加一个ListView。

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
    tools:context="com.example.listviewtest.MainActivity">

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"></ListView>

</RelativeLayout>
```

ListView有很多行,每一行是一个item。新建xml文件，用于设置item的布局。

这里设置每个item包括一个Button和一个TextView。

listview_item.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal" >

    <Button
        android:id="@+id/btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"/>

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:textSize="18sp"/>

</LinearLayout>
```

MainActivity.java

```

package com.example.listviewtest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.BaseAdapter;
import android.widget.Button;
import android.widget.ListView;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    private ListView mListView;
    private Button btn;
    public static String[] names = {"Android", "IPhone", "WindowsMobile", "Blackberry",
                        "WebOs", "Ubuntu", "Windows7", "Mac OS X"};

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mListView = (ListView) findViewById(R.id.listView);

        MyBaseAdapter mAdapter = new MyBaseAdapter();

        mListView.setAdapter(mAdapter);
        // ListView的item被点击监听
        mListView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                Toast.makeText(getApplicationContext(), names[position], Toast.LENGTH_SHORT).show();
            }
        });
    }

    class MyBaseAdapter extends BaseAdapter {
        // 获得item的个数
        public int getCount() {
            return names.length;
        }
        // 获取item的对象
        public Object getItem(int position) {
            return names[position];
        }
        // 获取item的id
        public long getItemId(int position) {
            return position;
        }
        // 得到item的view视图
        public View getView(final int position, View convertView, ViewGroup parent) {
            // 由于在不同的布局文件中，所以要通过inflate将listView_item.xml动态加载到activity_main.xml中。
            View view = View.inflate(MainActivity.this, R.layout.listview_item, null);
            TextView textView = (TextView) view.findViewById(R.id.textView);
            textView.setText(names[position]);

            Button btn = (Button) view.findViewById(R.id.btn);
            // 设置button没有焦点，否则点击listview会无效。
            btn.setFocusable(false);
            btn.setText(names[position]);
            btn.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    Toast.makeText(getBaseContext(), "btn " + names[position] + " pressed", Toast.LENGTH_SHORT).show();
                }
            });

            return view;
        }
    }
}
```

首先实现了BaseAdapter的类，实现了里面的几个抽象方法。

在getView()函数中设置了item中的视图，这样才会在activity_main.xml中显示出来。在textView中添加显示，添加button的监听事件，并且设置为没有焦点，ListView才能够获得焦点。否则只有点击button有限，点击ListView没有效果。

在OnCreate()中为ListView添加BaseAdapter对象。并未ListView添加了setOnItemClickListener()监听事件。

显示效果，点击button

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170313152958916-854762561.png)

点击ListView

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170313153104260-1993184266.png)


Adapter除了BaseAdapter，还有SimpleAdapter,ArrayAdapter。BaseAdapter使用范围广，SimpleAdapter适配Checkable，TextView，ImageView。ArrayAdapter只能适配TextView。


Tony Liu

2017-3-13, Shenzhen