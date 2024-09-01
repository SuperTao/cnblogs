
本文主要通过代码，了解Linux ELF文件的相关知识。

### ELF文件

ELF(Executable Linkable Format)是Linux平台的可执行文件。

ELF文件的类型：

* 可重定位类型

    * 包含代码和数据，用来链接成可执行文件或共享目标文件, linux的.o文件

* 可执行文件
    
    * 可以直接执行的程序

* 共享目标文件

    * .so文件

* 核心转储文件

    * core dump文件，系统意外时转储文件。

### 源码

**SimpleSection.c**

```
#include <stdio.h>
#include <stdlib.h>

// 全局变量，data区, 4字节
int global_int_var = 1;
// 全局变量，未初始化，bss区, 4字节
int global_uninit_var;

void func1(int i)
{
	// 里面有一个"%d\n"字符串,在rodata区, 共4字节
	printf("%d\n", i);
}

int main(void)
{
	// 静态变量，data区, 4字节
	static int static_var = 1;
	// 静态变量，未初始化，bss区, 4字节 
	static int static_var2;
	// 局部变量, 栈区
	int a = 1;
	// 局部变量, 栈区
	int b;
	// 静态变量，未初始化，64位系统，占用8字节
	static int *static_ptr = NULL;
	// 局部变量，栈区
	int *p;
	// malloc申请的空间，堆区
	p = malloc(sizeof(int));

	free(p);

	// 函数调用时，也会进行入栈出栈操作，一些局部变量都需要保存在占中
	func1(static_var + static_var2 + a + b);

	return 0;
}
```

### 编译文件 

只编译本文件方便分析，不连接其他的库。

编译使用的是ubuntu64位系统。

**gcc -c SimpleSection.c -o SimpleSection.o**

#### objdump查看段信息


**objdump -h SimpleSection.o**

```
#                    文件格式
SimpleSection.o:     file format elf64-x86-64
                  
Sections:
#                 大小      虚拟地址          逻辑地址          文件偏移  段边界对齐字数
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         0000007e  0000000000000000  0000000000000000  00000040  2**0
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE
  1 .data         00000008  0000000000000000  0000000000000000  000000c0  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  2 .bss          00000010  0000000000000000  0000000000000000  000000c8  2**3
                  ALLOC
  3 .rodata       00000004  0000000000000000  0000000000000000  000000c8  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .comment      00000027  0000000000000000  0000000000000000  000000cc  2**0
                  CONTENTS, READONLY
  5 .note.GNU-stack 00000000  0000000000000000  0000000000000000  000000f3  2**0
                  CONTENTS, READONLY
  6 .note.gnu.property 00000020  0000000000000000  0000000000000000  000000f8  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  7 .eh_frame     00000058  0000000000000000  0000000000000000  00000118  2**3
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, DATA

```

### 查看elf文件头

**readelf -h SimpleSection.o**

```
ELF Header:
# elf魔数
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
# 文件类型
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          1208 (bytes into file)
  Flags:                             0x0
# 文件头的大小
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
# 段表成员个数
  Number of section headers:         14
  Section header string table index: 13
```

### 查看elf段表

段表在ELF文件中位置有ELF文件头的"e_shoff"成员决定。

ELF文件的段结构就是由段表决定，编译器、链接器和装载器都是通过段表来定位和访问各个端的属性。

**readelf -S SimpleSection.o**

```
There are 14 section headers, starting at offset 0x4b8:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000000000  00000040
       000000000000007e  0000000000000000  AX       0     0     1
  [ 2] .rela.text        RELA             0000000000000000  00000368
       00000000000000a8  0000000000000018   I      11     1     8
  [ 3] .data             PROGBITS         0000000000000000  000000c0
       0000000000000008  0000000000000000  WA       0     0     4
  [ 4] .bss              NOBITS           0000000000000000  000000c8
       0000000000000010  0000000000000000  WA       0     0     8
  [ 5] .rodata           PROGBITS         0000000000000000  000000c8
       0000000000000004  0000000000000000   A       0     0     1
  [ 6] .comment          PROGBITS         0000000000000000  000000cc
       0000000000000027  0000000000000001  MS       0     0     1
  [ 7] .note.GNU-stack   PROGBITS         0000000000000000  000000f3
       0000000000000000  0000000000000000           0     0     1
  [ 8] .note.gnu.pr[...] NOTE             0000000000000000  000000f8
       0000000000000020  0000000000000000   A       0     0     8
  [ 9] .eh_frame         PROGBITS         0000000000000000  00000118
       0000000000000058  0000000000000000   A       0     0     8
  [10] .rela.eh_frame    RELA             0000000000000000  00000410
       0000000000000030  0000000000000018   I      11     9     8
  [11] .symtab           SYMTAB           0000000000000000  00000170
       0000000000000180  0000000000000018          12     9     8
  [12] .strtab           STRTAB           0000000000000000  000002f0
       0000000000000078  0000000000000000           0     0     1
  [13] .shstrtab         STRTAB           0000000000000000  00000440
       0000000000000074  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)
```

### 重定位表

有一个".rel.text"段，链接器在处理目标文件时，需要对目标文件中某些部位进行重定位，即代码段和数据段中那些对绝对地址的引用位置。

### 自定义段

通过**__attribute__((section("name")))**把变量或者函数放到以"name"作为段名的段中。

```
__attribute__((section(".var_section"))) 
int var_test;

__attribute__((section(".func_section"))) 
int func_test()
{   
	printf("helloworld\n");
}

```

通过objdump查看，可以发现多了两个段

```
  3 .var_section  00000004  0000000000000000  0000000000000000  000000c8  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  5 .func_section 0000001a  0000000000000000  0000000000000000  000000db  2**0
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE

```

#### 本文参考

《程序员的自我修养》

https://blog.csdn.net/SlowIsFastLemon/article/details/103593719
