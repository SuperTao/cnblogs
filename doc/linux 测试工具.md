最近在寻找linux的测试工具，试用了一些。记录如下。
### [memtester](http://pyropus.ca/software/memtester/)

内存测试工具，通过对内存进行读写进行测试。可以对同一块空间进行多次的读写。

源码分析

http://www.cnblogs.com/pslfree/p/4451091.html

http://my.oschina.net/liangzi1210/blog/113287?fromerr=sb56rgAm
交叉编译
```
vi conf-cc
vi conf-ld
把cc改为交叉编译器的名称，例如: arm-linux-gnueabihf-gcc
```
http://blog.csdn.net/dropping_1979/article/details/22865773

测试内存之前，使用free命令查看可用的内存。

测试效果
```
root@Tony:~# ./memtester 256M 1
memtester version 4.0.8 (32-bit)
Copyright (C) 2007 Charles Cazabon.
Licensed under the GNU General Public License version 2 (only).

pagesize is 4096
pagesizemask is 0xfffff000
want 256MB (268435456 bytes)
got  256MB (268435456 bytes), trying mlock ...locked.
Loop 1/1:
  Stuck Address       : ok         
  Random Value        : ok
  Compare XOR         : ok
  Compare SUB         : ok
  Compare MUL         : ok
  Compare DIV         : ok
  Compare OR          : ok
  Compare AND         : ok
  Sequential Increment: ok
  Solid Bits          : ok         
  Block Sequential    : ok         
  Checkerboard        : ok         
  Bit Spread          : ok         
  Bit Flip            : ok         
  Walking Ones        : ok         
  Walking Zeroes      : ok         

Loop 2/2:
  Bit Flip            : ok         
  Walking Ones        : ok         
  Walking Zeroes      : ok         
  ......
Done.
```


### [cpuburn-in](http://cpuburnin.com/)
测试cpu性能
目前只找到x86版本的。

http://linux-sunxi.org/Hardware_Reliability_Tests

### [cpuburn](https://launchpad.net/ubuntu/+source/cpuburn/1.4a-1)
使用汇编编写

http://linux.softpedia.com/get/System/Diagnostics/cpuburn-1407.shtml

### [stress](http://people.seas.harvard.edu/~apw/stress/)
压力测试工具

### stress-ng
压力测试工具

http://www.cyberciti.biz/faq/stress-test-linux-unix-server-with-stress-ng/

http://www.tecmint.com/linux-cpu-load-stress-test-with-stress-ng-tool/


### [iozone](http://www.iozone.org/)
测试硬盘，文件系统读写性能

移植方法

http://blog.sina.com.cn/s/blog_5d9051c00100dwfe.html