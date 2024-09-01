交叉编译qt的程序时，出现错误：error: icpc: Command not found。

解决方法，详情查看链接。

http://www.cnblogs.com/zengjfgit/p/4744507.html

更改project中的qmake参数,更改为交叉编译器名称，如下所示。

-spec qws/arm-linux-gnueabihf-g++