qt输入法不能用，ui中不能显示中文，开发板不能显示中文，这几个一直困扰这我，网上查找资料，在代码中添加各种支持，都没有解决问题。今天刚好解决了，记录于此。

### 参考链接 

---

　　http://blog.163.com/qimo601@126/blog/static/15822093201382611615112/

　　http://blog.csdn.net/willib/article/details/26397969

　　http://blog.csdn.net/zhangss415/article/details/7187202

### qtcreator无法输入中文

---

　　系统（Ubuntu）中已经安装了ibus拼音, qtcreator版本4.8.5。 

　　1. 安装ibus-qt4

　　　　sudo apt-get install ibus-qt4

　　2. 编辑profile文件:

　　　　vi ~/.profile

```
　　export XMODIFIERS="@im=ibus"
　　export GTK_IM_MODULE=ibus
　　export QT_IM_MODULE=xim
　　export ibus &
　　export LC_CTYPE=zh_CN.utf8
```

　　3. 系统重启

　　如果切换输入法的快捷键与qt冲突，记得更改。

### qt ui中不能显示中文

---

　　1. 终端输入 qtconfig

　　　　选择中文字体,Fangsong Ti或者 Song Ti

![](http://images2015.cnblogs.com/blog/745188/201607/745188-20160720134839966-1010056333.png)


　　　　保存并退出

　　2. 修改代码
```
　　程序main函数中添加
　　#include <QTextCodec>
　　
　　在QApplication a(argc, argv); 后添加
　　
　　QTextCodec::setCodecForTr(QTextCodec::codecForName("GB2312"));
　　QTextCodec::setCodecForTr(QTextCodec::codecForLocal());       //该语句可以解决在子窗口中的乱码问题！
```

### 开发板app中不能显示中文

---

　　简单粗暴方法,运行程序时，指定字体,例如运行名为io的可执行程序。

　　　　./io -fn unifont

　　还有一种方法式下载字体添加到开发板中，并修改代码，指定使用的字体,详情请留意参考链接。