在linux上调试wifi, 需要移植wireless tool进行验证，本文记录移植方法。

#### 参考链接

http://www.cnblogs.com/zengjfgit/p/5601473.html

http://blog.csdn.net/tigerjibo/article/details/12784901

#### 下载地址

https://hewlettpackard.github.io/wireless-tools/Tools.html

#### 移植方法

下载源码，修改makefile

```
## Compiler to use (modify this for cross compile).
CC = arm-linux-gcc
## Other tools you need to modify for cross compile (static lib only).
AR = arm-linux-ar
RANLIB = arm-linux-ranlib

```
编译:  make

将生成的libiw.so.29放到板子的/lib目录

将生成的iwlist,iwconfig等添加执行权限，放到/sbin/,/bin/等PATH目录。

---

Tony Liu

2016-12-5, Shenzhen