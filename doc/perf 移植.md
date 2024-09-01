## perf 移植

perf工具用于系统性能的调优，程序优化。源码在kenel/tools/perf目录。
我在imx6平台上进行移植。将自己的移植过程记录如下。

### 参考链接
http://blog.csdn.net/vc66vcc/article/details/51437853

http://blog.csdn.net/vc66vcc/article/details/51436569

http://albert-oma.blogspot.com/2015/03/embedded-arm-in-linux-perf.html

https://sites.google.com/site/hhmasterthesis/project-updates/2012-06-02

http://blog.csdn.net/mtofum/article/details/44108601

### 下载地址
[zlib](http://zlib.net/)
[elfutils](https://kojipkgs.fedoraproject.org/packages/elfutils/)

### 注意

编译使用的编译器要与内核是同一个。
编译安装要用到gcc的路径，这里生成一个环境变量。

export CROSS_COMPILE_DIR=/home/Qt/arm-linux-gcc/gcc-4.6.2-glibc-2.13-linaro-multilib-2011.12/fsl-linaro-toolchain/

### kernel配置
首先查看.config, kernel是否打开了相关的设置。
CONFIG_PERF_EVENTS=y
CONFIG_HW_PERF_EVENTS=y 
若不支持，需要从新编译kernel，设置方式如下
kernel/linux-2.6.38$ make menuconfig
  Symbol: HW_PERF_EVENTS [=n]
  Location:
   -> Kernel Features
   Symbol: PERF_EVENTS [=n]
  Location:
  -> General setup
   -> Kernel Performance Events And Counters


### zlib编译与安装
版本 zlib-1.2.8
```
CC=arm-linux-gcc ./configure --prefix=$CROSS_COMPILE_DIR/libc/usr/
make
make install
```
### elfutils编译与安装
版本 elfutils-0.148
```
./configure --host=arm-linux --prefix=$CROSS_COMPILE_DIR/libc/usr/ --exec-prefix=$CROSS_COMPILE_DIR/libc/usr/
vi Makefile 更改
```
修改Makefile
```
SUBDIRS = config m4 lib libelf libebl libdwfl libdw libcpu libasm backends \
      src po tests
删除 libcpu
SUBDIRS = config m4 lib libelf libebl libdwfl libdw libasm backends \
      src po tests
```

vi backends/Makefile 更改, 删除有关i386, x86_64的条目，如下：
```
原来
libebl_pic = libebl_i386_pic.a libebl_sh_pic.a libebl_x86_64_pic.a \
        libebl_ia64_pic.a libebl_alpha_pic.a libebl_arm_pic.a \
        libebl_sparc_pic.a libebl_ppc_pic.a libebl_ppc64_pic.a \
        libebl_s390_pic.a
删除后
libebl_pic = libebl_sh_pic.a \
        libebl_ia64_pic.a libebl_alpha_pic.a libebl_arm_pic.a \
        libebl_sparc_pic.a libebl_ppc_pic.a libebl_ppc64_pic.a \
        libebl_s390_pic.a

如下进行注释
noinst_LIBRARIES = $(libebl_pic)
noinst_DATA = $(libebl_pic:_pic.a=.so)
libelf = ../libelf/libelf.so
#libelf = ../libelf/libelf.a
libdw = ../libdw/libdw.so
#libdw = ../libdw/libdw.a
#i386_SRCS = i386_init.c i386_symbol.c i386_corenote.c i386_cfi.c \
#       i386_retval.c i386_regs.c i386_auxv.c i386_syscall.c
#cpu_i386 = ../libcpu/libcpu_i386.a
#libebl_i386_pic_a_SOURCES = $(i386_SRCS)
#am_libebl_i386_pic_a_OBJECTS = $(i386_SRCS:.c=.os)
sh_SRCS = sh_init.c sh_symbol.c sh_corenote.c sh_regs.c sh_retval.c
libebl_sh_pic_a_SOURCES = $(sh_SRCS)
am_libebl_sh_pic_a_OBJECTS = $(sh_SRCS:.c=.os)
#x86_64_SRCS = x86_64_init.c x86_64_symbol.c x86_64_corenote.c x86_64_cfi.c \
#x86_64_retval.c x86_64_regs.c i386_auxv.c x86_64_syscall.c
#cpu_x86_64 = ../libcpu/libcpu_x86_64.a
#libebl_x86_64_pic_a_SOURCES = $(x86_64_SRCS)
#am_libebl_x86_64_pic_a_OBJECTS = $(x86_64_SRCS:.c=.os)
#libebl_i386.so: $(cpu_i386)
#libebl_x86_64.so: $(cpu_x86_64)
```

make会出错，删除下面三个文件的-Werror，
lib/Makefile
libasm/Makefile
src/Makefile
例如：
```
原来
AM_CFLAGS = -std=gnu99 -Wall -Wshadow $(if \
    $($(*F)_no_Werror),,-Werror) $(if \
    $($(*F)_no_Wunused),,-Wunused -Wextra) $(if \
    $($(*F)_no_Wformat),-Wno-format,-Wformat=2) $($(*F)_CFLAGS) \
    $(am__append_1) -fpic
更改后
AM_CFLAGS = -std=gnu99 -Wall -Wshadow $(if \
    $($(*F)_no_Werror),,) $(if \
    $($(*F)_no_Wunused),,-Wunused -Wextra) $(if \
    $($(*F)_no_Wformat),-Wno-format,-Wformat=2) $($(*F)_CFLAGS) \
    $(am__append_1) -fpic
make
make install
```

### 编译perf
修改顶层Makefile
```
CFLAGS = -fno-omit-frame-pointer -ggdb3 -Wall -Wextra -std=gnu99 $(CFLAGS_OPTIMIZE) -D_FORTIFY_SOURCE=2 $(EXTRA_WARNINGS) $(EXTRA_CFLAGS)
EXTLIBS = -lpthread -lrt -lelf -lm -lebl -ldl -L/home/Qt/arm-linux-gcc/gcc-4.6.2-glibc-2.13-linaro-multilib-2011.12/fsl-linaro-toolchain/libc/usr/lib
ALL_CFLAGS = $(CFLAGS) -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE -I/home/Qt/arm-linux-gcc/gcc-4.6.2-glibc-2.13-linaro-multilib-2011.12/fsl-linaro-toolchain/libc/usr/include
```
进行编译
```
make LDFLAGS=-static ARCH=arm CROSS_COMPILE=arm-linux- DEBUG=1 LIBDW_DIR=$CROSS_COMPILE_DIR/libc/usr/ HAVE_CPLUS_DEMANGLE=1
```
生成perf可执行文件更新到板上。