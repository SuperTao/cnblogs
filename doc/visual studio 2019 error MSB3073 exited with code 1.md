在用vs2019编译C++工程时，出现错误。

原因是设置的命令没有运行成功，需要将这条命令关闭。不编译就行了。

解决方法，打开property manager，打开property pages，将其中的Post-Build Event改成No。

操作方法View->Other windows->Property Manager->右键工程，选择Properties->Build Events->Post-Build Event。

将其中的Use In Build更改为No。

参考连接：

https://blog.csdn.net/wangmengmeng99/article/details/68926030

https://blog.csdn.net/zhounanzhaode/article/details/50373105