```
参考链接：
	https://blog.csdn.net/qq_39759656/article/details/82858101

为什么没有core文件
生成core文件的条件有3点：
1.             limit大小 够  
2.             执行文件没有strip  
3.             存放的目录有写权限。路径在/proc/sys/kernel/core_pattern里面。 
	Exam:
	echo "/var/tmp/core-%e-%p-%t" > /proc/sys/kernel/core_pattern
	echo "/nfs/core-%e-%p-%t" > /proc/sys/kernel/core_pattern
	

一般来说GDB主要调试的是C/C++的程序。要调试C/C++的程序，首先在编译时，我们必须要把调试信息加到可执行文件中。
使用编译器（cc/gcc/g++）的 -g 参数可以做到这一点。

$ echo stop > /proc/dh_watchdog //关闭看门狗
$ echo "/home/core-%e-%p-%t" >/proc/sys/kernel/core_pattern
$ ulimit -c unlimited 

查看调用栈
arm-ca9-linux-gnueabihf-gdb bttest core-bttest-798-1621601170

(gdb) bt
#0  0x76e09688 in __memcpy_by4 () from /lib/libc.so.6
#1  0x76ddc450 in _IO_vfscanf_internal () from /lib/libc.so.6
Backtrace stopped: previous frame inner to this frame (corrupt stack?)
(gdb) f 1
#1  0x76ddc450 in _IO_vfscanf_internal () from /lib/libc.so.6
(gdb) sp
Undefined command: "sp".  Try "help".
(gdb) x /100a $sp
```