在调试ui的时候，需要将背景色变为透明，与母控件的颜色一致，并且除去边框。

参考链接：

　　http://www.qtcentre.org/threads/12148-how-QTextEdit-transparent-to-his-parent-window

```
除去背景色，使透明。

    ui->textBrowser->viewport()->setAutoFillBackground(false);

控件边框。

    ui->textBrowser->setFrameStyle(QFrame::NoFrame)
```

Tony Liu

2016-11-3, Shenzhen