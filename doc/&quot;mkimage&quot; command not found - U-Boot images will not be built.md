编译内核的时候出现错误："mkimage" command not found - U-Boot images will not be built

参考链接

http://blog.csdn.net/wen0605/article/details/8448907

make uImage的时候要用到uboot/tool/mkimage工具。

将mkimage放到/usr/bin等目录中即可或者将mkimage的路径添加到环境变量中。

Tony Liu

2016-12-15，Shenzhen