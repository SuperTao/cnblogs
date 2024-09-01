使用RichTextBox控件用于显示数据时，滚动条只停留在开头，而我希望能够一直更新，显示最后一行的内容。解决方法记录于此。

转载自以下链接：

  　　http://blog.csdn.net/xelone/article/details/5916451

```
    //让文本框获取焦点，不过注释这行也能达到效果
    richTextBoxReceive.Focus();
    //设置光标的位置到文本尾   
    richTextBoxReceive.Select(richTextBoxReceive.TextLength, 0);
    //滚动到控件光标处   
    richTextBoxReceive.ScrollToCaret();
    richTextBoxReceive.AppendText(text);
```

Tony Liu

2016-9-15, Shenzhen