android创建xml文件的方法。

要操作android的外部存储，所以要在AndroidManifest.xml文件中添加权限。

`<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />`

首先要创建一个XmlSerialize对象。

并创建一个文件，并打开获得输出流。

使用XmlSerilaize中的操作方法向输出流中写入数据。

```
package com.example.xmltest;

import android.os.Environment;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Xml;
import android.widget.Toast;

import org.xmlpull.v1.XmlSerializer;

import java.io.File;
import java.io.FileOutputStream;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Serializer();
    }

    public void Serializer() {

        try {
	    // 获取外部存储路径,并创建文件
            File file = new File(Environment.getExternalStorageDirectory(), "person.xml");
	    // 创建一个文件输出流
            FileOutputStream os = new FileOutputStream(file);
            XmlSerializer serializer = Xml.newSerializer();            
            // 设置输出的编码格式
            serializer.setOutput(os, "UTF-8");
            // 设置文档的编码格式，这个会在xml中显示出来
            serializer.startDocument("UTF-8", true);
			// 添加注释
            serializer.comment("comment, xml file test");
            // 开始标签
            serializer.startTag(null, "tag_1");
            // 标签对应的内容
            serializer.text("hello");
            // 结束
            serializer.endTag(null, "tag_1");

            serializer.startTag(null, "tag_2");
            // 属性
            serializer.attribute(null, "id", "2");
            serializer.text("hello");
            serializer.endTag(null, "tag_2");

            serializer.startTag(null, "tag_3");
            // 在标签中再设置一个标签
            serializer.startTag(null, "tag_3_3");
            serializer.attribute(null, "id", "3");
            serializer.text("hello");
            serializer.endTag(null, "tag_3_3");
            serializer.endTag(null, "tag_3");

            serializer.endDocument();        // 文档结束
            serializer.flush();

            os.close();                    // 关闭输出流
            Toast.makeText(this, "success", Toast.LENGTH_SHORT).show();
        } catch (Exception e) {
            e.printStackTrace();
            Toast.makeText(this, "fail", Toast.LENGTH_SHORT).show();
        }
    }
}
```


输出效果

```
<?xml version='1.0' encoding='UTF-8' standalone='yes' ?>
<!--comment, xml file test-->
<tag_1>hello</tag_1>
<tag_2 
	id="2">hello
</tag_2>
<tag_3>
	<tag_3_3 
		id="3">hello
	</tag_3_3>
</tag_3>
```

Tony Liu

2017-3-20, Shenzhen