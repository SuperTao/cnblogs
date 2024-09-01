每次QString转换int或者char的时候都要查资料，记录一下，方便下次查看。

#### 参考：

　　http://blog.csdn.net/ei__nino/article/details/7297791

　　http://www.cnblogs.com/Romi/archive/2012/03/12/2392478.html

#### QString 转 char

```
Qstring  str;

char*  ch;

QByteArray ba = str.toLatin1();    

ch=ba.data();
```

#### 16进制的QSting转成int

遇到例如'0xFF','0XFF'的QString

```
QString addr_s = ui->lineEdit_addr->text();
unsigned char addr ;
bool ok;
//判断是否是'0x'或者'0X'开头
if (addr_s.startsWith("0x") || addr_s.startsWith("0X"))
{
    QString addr_t = addr_s.mid(2);                 //QString截取，从索引值为2的位置开始
    addr = (unsigned char)addr_s.toInt(&ok, 16);    //转成16进制
}
else
{
    // 10进制直接转化
    addr = addr_s.toInt();
}
```

#### int转QString

```
long a = 63;  
QString s = QString::number(a, 10);             // s == "63" , 转成10进制 
QString t = QString::number(a, 16).toUpper();     // t == "3F" , 转16进制 
```

Tony Liu

2016-9-24, Shenzhen