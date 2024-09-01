最近用到了QMessgaeBox和QInputDialog，QMessageBox用于提示，警告等消息，QInputDialog给用户弹出输入对话框。

#### 参考链接

http://chenboqiang.blog.51cto.com/1068057/321900

http://blog.csdn.net/xgbing/article/details/7828212

http://devbean.blog.51cto.com/448512/219338/

#### QMessageBox

```
  QMessageBox::StandardButton reply;
    reply = QMessageBox::warning(NULL, 			// parent
								"warning", 		// title
								"Are you sure to erase the EMMC?",	// text
                                 QMessageBox::Yes|QMessageBox::No, 	// 有几个选项，这里有YES,和NO
								 QMessageBox::No);					// 默认选项是NO
    if (reply == QMessageBox::Yes)	//选项的判断
    {
		...
		...
    }
    else if (reply == QMessageBox::No)	
	{
		...
		...
	}
```
	
#### QInputDialog

```
    QString text = QInputDialog::getText(NULL, "Password",
                                        "Please input password",
                                        //	QLineEdit::Password,	//输入的是密码，不显示明文
                                        QLineEdit::Normal,			//输入框明文
                                        NULL,
                                        &isOK);

    if(isOK) {
           QMessageBox::information(NULL, "Information",
                                               "Your comment is: <b>" + text + "</b>",
                                               QMessageBox::Yes | QMessageBox::No,
                                               QMessageBox::Yes);
    }
```

Tony Liu

2016-11-14, Shenzhen