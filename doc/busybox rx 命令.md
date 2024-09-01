rx命令使用xmodem传送文件，只需要串口线就传送。

在文件系统输入如下命令,传送文件到板子上，板子上保存文件的名称为file

rx file

在secureCRT中选择Transfer->Send Xmodem。选择要传送的文件即可。

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161026111431093-1085212777.png)

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161026111443703-1751100490.png)

```
root@freescale ~$ rx file
CCC
Starting xmodem transfer.  Press Ctrl+C to cancel.
Transferring a.txt...
  100%      51 bytes   51 bytes/sec 00:00:01       0 Errors  
```

Tony Liu

2016-10-26, Shenzhen