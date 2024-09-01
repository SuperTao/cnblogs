Arraylist相当于动态数组，可以动态的添加或者删除其中的元素。

参考链接

http://beginnersbook.com/2013/12/java-arraylist/

```
package com.example.arraylisttest;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ArrayList<String> obj = new ArrayList<String>();
		// 添加
        obj.add("one");
        obj.add("two");
        obj.add("three");
        obj.add("four");
        Log.d("debug", obj + "");

		// 往指定索引添加
        obj.add(0, "five");
        obj.add(1, "six");
        Log.d("debug", obj + "");

		// 删除指定索引
        obj.remove(1);
        Log.d("debug", obj + "");
		
		// 删除指定元素
        obj.remove("three");
        Log.d("debug", obj + "");

		// 更改
        obj.set(0, "seven");
        Log.d("debug", obj + "");
    }
}
```

输出结果

```
03-20 19:58:36.528 5490-5490/? D/debug: [one, two, three, four]
03-20 19:58:36.528 5490-5490/? D/debug: [five, six, one, two, three, four]
03-20 19:58:36.529 5490-5490/? D/debug: [five, one, two, three, four]
03-20 19:58:36.529 5490-5490/? D/debug: [five, one, two, four]
03-20 19:58:36.529 5490-5490/? D/debug: [seven, one, two, four]
```

Tony Liu

2017-3-20, Shenzhen