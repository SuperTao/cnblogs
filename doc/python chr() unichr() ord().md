了解一下python chr()，unichr(),ord()函数的用法。

#### 参考链接：

　　http://crazier9527.iteye.com/blog/411001 

#### chr() 

输入参数(取值范围0-255)整数,输出整为ascii码

```
>>> chr(0)
'\x00'
>>> chr(65)
'A'
>>> chr(127)
'\x7f'
>>> chr(126)
'~'
>>> chr(255)
'\xff'
```

#### unichr() 

输入整数，输出为unicode

参数范围依赖于Python是如何被编译的。如果是配置为USC2的Unicode，那么它的允许范围就是range（65536）或0x0000-0xFFFF；如果配置为UCS4，那么这个值应该是range（1114112）或0x000000-0x110000。

```
>>> unicode(0)
u'0'
>>> unicode(65)
u'65'

```

#### ord() 

输入参数ascii字符或者unicode字符。输出整数

```
>>> ord('0')
48
>>> ord('A')
65
>>> ord(u'\uffff')
65535
>>> ord(u'a')
97
>>> ord(u'A')
65
>>> ord(u'\u2020')
8224
```

Tony Liu

2016-9-22, Shenzhen