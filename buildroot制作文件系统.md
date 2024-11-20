# buildroot制作文件系统

参考文档：
	https://buildroot.org/downloads/manual/manual.html

buildroot除了可以制作文件系统。编译一些工具也是很方便的。

网上需要使用一些工具，经常需要下载很多的依赖库，交叉编译非常麻烦。

buildroot可以将所有的以来工具都下载好。对于可以执行文件，还可以选择编译成动态连接，或者是静态编译。

# 配置Toolchain

https://www.51cto.com/article/667736.html

make menuconfig

make

# 编译busybox

sudo make busybox-menuconfig

默认不选择uboot和kernel

# 编译工具

https://www.cnblogs.com/erhu-67786482/p/13665042.html

可执行文件默认依赖动态库。

如果需要静态编译。需要在menuconfig中将BR2_STATIC_LIBS选上。

以编译tcpdump为例。

在make menuconfig中查找tcpdump。选上之后。

make tcpdump
