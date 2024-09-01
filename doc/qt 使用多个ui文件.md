一般的QT工程只有一个ui，本文记录如何在一个工程中使用多个ui文件。

参考链接：

http://www.cnblogs.com/lc-cnblong/p/3182903.html    
    

创建方法,鼠标在工程名处右键：

add New -> Qt -> Qt Designer Form Class -> Widget

就创建了一个新类，并且里面实现了ui。

注：

在最后要选择Widget，而不能选择Main Window,好像会跟自己定义的MainWindow冲突。

我出现这种情况，将MainWindows更改为widget之后，编译通过。