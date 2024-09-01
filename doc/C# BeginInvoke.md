在用C#编写串口助手时，希望创建线程更新UI，网上有人采用BeginInvoke方法, 这里记录一下使用方法。

#### 参考链接：

　　http://blog.csdn.net/zaijzhgh/article/details/14104693

　　http://www.cnblogs.com/wangshenhe/archive/2012/05/25/2516842.html

　　https://msdn.microsoft.com/en-us/library/2e08f6yc(v=vs.110).aspx

```
    // 定义委托, 我的理解，相当于C中的函数指针
    delegate void SetTextCallback(String text);

    ......
    String data = serialPort.ReadExisting();
    if (data != String.Empty)
    {
        // 创建线程, 线程中的调用函数SetText, 这里相当于回调函数, 参数 data
        this.BeginInvoke(new SetTextCallback(SetText), new object[] { data });
        // 创建之后，使用EndInvoke会诸塞调用线程，所以这里没有调用
    }

    // 线程中调用的函数
    private void SetText(String text)
    {
            richTextBoxReceive.AppendText(text);
    }
        
```

Tony Liu

2016-9-17, Shenzhen