```
1. Makefile规则

target: prerequisites
	command
	
目标文件：依赖文件
	编译的命令

2. 举例学习
	
2.1 在同一个目录创建三个文件

$ tree
.
+-- func1.c
+-- func1.h
+-- func2.c
+-- func2.h
+-- main.c
+-- Makefile

func1.c
----------------------------------
#include <stdio.h>
#include "func1.h"

void func1()
{
	printf("%s\n", __func__);
}
----------------------------------

func1.h
----------------------------------
#ifndef _FUNC1_H_
#define _FUNC1_H_

void func1(void);

#endif
----------------------------------

func2.c
----------------------------------
#include <stdio.h>
#include "func1.h"

void func2()
{
	printf("%s\n", __func__);
}
----------------------------------

func2.h
----------------------------------
#ifndef _FUNC1_H_
#define _FUNC1_H_

void func2(void);

#endif
----------------------------------

main.c
----------------------------------
#include "func1.h"
#include "func2.h"

int main(void)
{
	func1();
	func2();
	
	return 0;
}
----------------------------------

Makefile
----------------------------------
# 连接生成可执行文件，用-o指定生成的文件名称
main: main.o func1.o func2.o
	gcc -o main main.o func1.o func2.o

# 编译生成.o目标文件
main.o: main.c
	gcc -c main.c

func1.o: func1.c func1.h
	gcc -c func1.c

func2.o: func2.c func2.h
	gcc -c func2.c

clean:
	rm main *.o
----------------------------------
make输出结果：
gcc -c main.c
gcc -c func1.c
gcc -c func2.c
gcc -o main main.o func1.o func2.o


2.2 更改层次结构,将.c和.h分别放到src和inc目录。
$ tree
.
+-- inc
¦   +-- func1.h
¦   +-- func2.h
+-- main.c
+-- Makefile
+-- src
    +-- func1.c
    +-- func2.c

Makefile
----------------------------------
# 链接生成可执行文件
main: main.o src/func1.o src/func2.o
	gcc -I./inc -o main main.o src/func1.o src/func2.o 

# -c生成.o目标文件
main.o: main.c
	gcc -I./inc -c main.c

# 生成目标文件，用-o指定放在src目录
# 目标文件和依赖文件以及编译的时候，都需要添加路径
src/func1.o: src/func1.c
	gcc -I./inc -o src/func1.o -c src/func1.c

src/func2.o: src/func2.c
	gcc -I./inc -o src/func2.o -c src/func2.c

clean:
	rm main main.o src/func1.o src/func2.o 
----------------------------------
输出结果
gcc -I./inc -c main.c
gcc -I./inc -o src/func1.o -c src/func1.c
gcc -I./inc -o src/func2.o -c src/func2.c
gcc -I./inc -o main main.o src/func1.o src/func2.o

另外一种写法
----------------------------------
SRC_PATH += .
SRC_PATH += ./src

CFLAGS += -I./inc

C_FILES += $(foreach dir, $(SRC_PATH), $(wildcard $(dir)/*.c))
# 通过warning函数打印结果
$(warning C_FILES: $(C_FILES))
OBJS = $(patsubst %.c, %.o, $(C_FILES))
$(warning OBJS   : $(OBJS))

main: $(OBJS)
	$(CC) -o main $(OBJS)
	
clean:
	rm main $(OBJS)
----------------------------------
输出结果
Makefile:8: C_FILES:  ./main.c  ./src/func1.c ./src/func2.c
Makefile:10: OBJS   :  ./main.o  ./src/func1.o  ./src/func2.o
cc -I./inc   -c -o main.o main.c
cc -I./inc   -c -o src/func1.o src/func1.c
cc -I./inc   -c -o src/func2.o src/func2.c
cc -o main  ./main.o  ./src/func1.o  ./src/func2.o

# $(foreach <var>,<list>,<text>)
# 这个函数的意思是，把参数<list>;中的单词逐一取出放到参数<var>;所指定的变量中，然后再执行< text>;所包含的表达式。每一次<text>;会返回一个字符串，循环过程中，<text>;的所返回的每个字符串会以空格分隔，

# wildcart把指定目录的.c文件全部展开
	
#格式：$(patsubst pattern,replacement,text)
#名称：模式字符串替换函数——patsubst。
#功能：查找text中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式pattern，如果匹配的话，则以replacement替换。
```