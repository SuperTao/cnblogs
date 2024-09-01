```
# 文件结构
#$ tree
#.
#+-- dir1
#¦   +-- 1.c
#¦   +-- a.s
#+-- dir2
#    +-- 11.c
#    +-- aa.s
#+-- Makefile

------------------------------------------------------------------------------------
# Makefile
# wildcard展开后面通配符的内容
SRCS1 += $(wildcard dir1/*)		# 获取第一个目录的所有文件
SRCS2 += $(wildcard dir2/*)		# 获取第二个目录的所有文件


SRCS += $(notdir $(SRCS1))		# 去掉路径
SRCS += $(notdir $(SRCS2))


filter_test1 += $(filter %.c, $(SRCS))          # 过滤所有的.c文件
filter_test2 += $(filter %.s, $(SRCS))          # 过滤所有的.c文件
filter_test3 += $(filter %.c %.s, $(SRCS))		# 过滤.c和.s文件

patsubst_test += $(patsubst %.c,%.o, $(filter_test1))	# 过滤c文件，替换成.o
patsubst_test += $(patsubst %.s,%.o, $(filter_test2))	# 过滤s文件，替换成.o


file1 = $(foreach n, $(SRCS), $(n))	# 循环打印文件内容
file2 = $(foreach n, $(SRCS), $(n).o)	# 循环打印文件内容

all:
	@echo $(SRCS1)
	@echo $(SRCS2)
# 打印两个目录的所有文件
	@echo $(SRCS)
# 过滤所有的c文件
	@echo $(filter_test1)
# 过滤所有的s文件
	@echo $(filter_test2)
# 过滤c文件和s文件
	@echo $(filter_test3)
# 替换后缀的.c为.o
	@echo $(patsubst_test)
# for循环打印所有文件
	@echo $(file1)
	@echo $(file2)

------------------------------------------------------------------------------
# 输出结果：
dir1/a.s dir1/1.c
dir2/11.c dir2/aa.s
a.s 1.c 11.c aa.s
1.c 11.c
a.s aa.s
a.s 1.c 11.c aa.s
1.o 11.o a.o aa.o
a.s 1.c 11.c aa.s
a.s.o 1.c.o 11.c.o aa.s.o
#notdir: 去除路径
#wildcard函数：扩展通配符
#
#foreach的作用：
#	$(foreach <var>,<list>,<text>)
#	这个函数的意思是，把参数<list>;中的单词逐一取出放到参数<var>;所指定的变量中，然后再执行< text>;所包含的表达式。
#	每一次<text>;会返回一个字符串，循环过程中，<text>;的所返回的每个字符串会以空格分隔，最后当整个循环结束时，
#	<text>;所返回的每个字符串所组成的整个字符串（以空格分隔）将会是foreach函数的返回值。
#	所以，<var>;最好是一个变量名，<list>;可以是一个表达式，而<text>;中一般会使用<var>;这个参数来依次枚举<list>;中的单词。
#
#patsubst函数：
#格式： $(patsubst <pattern>,<replacement>,<text> ) 
#名称：模式字符串替换函数——patsubst。
#功能：查找<text>中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式<pattern>，如果匹配的话，则以<replacement>替换。
#　　　这里，<pattern>可以包括通配符“%”，表示任意长度的字串。如果<replacement>中也包含“%”，
#      那么，<replacement>中的这个“%”将是<pattern>中的那个“%”所代表的字串。
#　　　（可以用“\”来转义，以“\%”来表示真实含义的“%”字符）
#返回：函数返回被替换过后的字符串。
#filter函数：
#$(filter PATTERN…,TEXT) 
#函数名称：过滤函数—filter。 
#函数功能：过滤掉字串“TEXT”中所有不符合模式“PATTERN”的单词，保留所
#有符合此模式的单词。可以使用多个模式。模式中一般需要包含模式字
#符“%”。存在多个模式时，模式表达式之间使用空格分割。 
#返回值：空格分割的“TEXT”字串中所有符合模式“PATTERN”的字串。 
#函数说明：“filter”函数可以用来去除一个变量中的某些字符串，
```