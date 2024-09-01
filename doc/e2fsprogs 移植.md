e2fsprogs是用维护ext2，ext3和ext4文件系统的工具程序集。检测和修复文件系统，需要用到其中的fsck, ext2fs等工具，

由于开发板上没有，重新制作文件系统又比较麻烦。所以就需要自己移植。本文记录移植方法。

### 参考文档 

http://blog.csdn.net/crazycoder8848/article/details/8510775

https://zh.wikipedia.org/wiki/E2fsprogs

### 下载地址

https://sourceforge.net/projects/e2fsprogs/files/e2fsprogs/

### 编译方法

我下载的版本e2fsprogs-1.42

可以编译成静态库和动态库2种，视自己情况而定。

本文只记录编译静态库的方法，编译成动态库的方式请查看参考链接。

```
tar -xzf e2fsprogs-1.42tar.gz

cd e2fsprogs-1.42

mkdir release

cd release/

编译成静态库方法:

../configure --host=arm-linux CC=arm-linux-gcc LDFLAGS=-static

make

生成的文件在release目录中，将需要的可执行文件添加到开发板中即可。
```