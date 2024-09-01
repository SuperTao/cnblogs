使用XmlPullParser解析xml文件。

要解析的xml文件如下所示。

weather.xml

```
<?xml version="1.0" encoding="utf-8"?>
<info>
    <city id="1">
        <name>Shenzhen</name>
        <temp>30</temp>
    </city>
    <city id="2">
        <name>Beijing</name>
        <temp>20</temp>
    </city>
</info>
```

XmlPullParser的next()移动到下一个节点。

XmlPullParser的nextText()获取节点的值。

getAttributeValue(null, "id")根据属性名称获取节点中属性的值.第一个参数一般是null，第二个参数是属性名称。

MainActivity.java如下所示。

```
package com.example.xmlpullparsertest;

import android.content.res.XmlResourceParser;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.widget.Toast;

import org.xmlpull.v1.XmlPullParser;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        WeatherInfo weatherInfo = null;
        // 保存多个节点的列表
        List<WeatherInfo> infos = null;
        // 获取xml文件
        XmlPullParser xmlPullParser = getResources().getXml(R.xml.weather);
        try {
            //getEventType()方法返回XMLStreamConstants接口中定义的一个标记常量，表示当前指针所指向标记（或事件）的类型
            int mEventType = xmlPullParser.getEventType();
            while (mEventType != XmlPullParser.END_DOCUMENT) {
                switch (mEventType) {
                    case XmlPullParser.START_DOCUMENT:
                        break;
                    case XmlPullParser.START_TAG:
						// 判断是否是"info"节点
                        if ("info".equals(xmlPullParser.getName())) {
                            infos = new ArrayList<WeatherInfo>();
                        }
                        else if ("city".equals(xmlPullParser.getName())) {
                            weatherInfo = new WeatherInfo();
                            // 通过属性名称获取节点属性的值
							// 获取"city"节点的”id“属性的值
                            String id = xmlPullParser.getAttributeValue(null, "id");
                            weatherInfo.setId(Integer.parseInt(id));
                        }
                        else if ("name".equals(xmlPullParser.getName())) {
                            // 获取节点的值
                            String str = xmlPullParser.nextText();
                            weatherInfo.setName(str);
                        }
                        else if ("temp".equals(xmlPullParser.getName())) {
                            // 后去temp节点对应的值
                            String str = xmlPullParser.nextText();
                            weatherInfo.setTemp(str);
                        }
                        break;
                    case XmlPullParser.END_TAG:
                        // 根据END_TAG，将获取的信息添加到Array中
                        if ("city".equals((xmlPullParser.getName()))) {
                            infos.add(weatherInfo);
                            weatherInfo = null;
                        }
                        break;
                }
                // 移动到下一个节点,继续解析
                mEventType = xmlPullParser.next();
            }
        } catch (Exception e) {
            Toast.makeText(this, "fail", Toast.LENGTH_SHORT);
        }
		// 查看
        for(WeatherInfo tmp:infos) {
            Toast.makeText(this, tmp.getId() + " " + tmp.getName()+ " " + tmp.getTemp(), Toast.LENGTH_SHORT).show();
        }
    }
}
```

Tony Liu

2017-3-20, Shenzhen