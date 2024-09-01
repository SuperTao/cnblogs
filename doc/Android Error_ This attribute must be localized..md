在android中使用mmm命令编译程序是出现错误。

这种问题一般情况是因为在res/xml文件夹下的中, 或者在res/layout下的文件中出现了没有多语言话的文本例.

解决方法：

不直接在布局文件中写字符串，而是在res/values/strings.xml中写。

例如res/values/strings.xml

`<string name="topLeftContent">TOP_LEFT</string>`

布局文件中再引用

`android:text="@string/topLeftContent"`

Tony Liu

2017-3-23, Shenzhen