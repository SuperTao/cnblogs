```
/*
  1)定义一个va_list类型的变量，变量是指向参数的指针；
  2)va_start初始化刚定义的变量，第二个参数是最后一个显式声明的参数；
  3)va_arg返回变长参数的值，第二个参数是该变长参数的类型；
  4)va_end将a)定义的变量重置为NULL。
 */

#include <stdio.h>
#include <stdarg.h>
#include <string.h>

void printf1(int num, ...)
{
	int i;
	// 定义一个
	va_list args;
	// 初始化va_list， 第二个参数是最后显示定义的参数
	va_start(args, num);
	for (i = 0; i < num; i++)
		printf("%d ", va_arg(args, int));
	printf("\n");
	va_end(args);
}

void printf2(int num, int count, ...)
{
	va_list args;
	int i; 
        // 最后一个显式声明的参数是count
	va_start(args, count);
	//va_start(args, count);
	for (i = 0; i < num; i++)
		printf("%d ", va_arg(args, int));
	printf("\n");
	va_end(args);
}


enum {
	MSG_EXCESSIVE, MSG_MSGDUMP, MSG_DEBUG, MSG_INFO, MSG_WARNING, MSG_ERROR
};

int wpa_debug_level = MSG_INFO;

void wpa_printf(int level, const char *fmt, ...)
{
	va_list ap;
	// 最后一个显式声明的参数是fmt
	va_start(ap, fmt);

	if (level > wpa_debug_level)
	{
		// 将可变参数列表的格式化数据打印到stdout
		vprintf(fmt, ap);
                // 可以根据需要，将数据写入文件中
	}

	va_end(ap);
}


int main(int argc, char **argv)
{
	printf1(3, 1, 2, 3);

	printf2(3, 4, 'a', 'b', 'c', 'd');	

	wpa_printf(MSG_ERROR, "hello world, %s\n", "Tao");
	wpa_printf(MSG_WARNING, "nice to meet you, %s\n", "Tao");
	wpa_printf(MSG_DEBUG, "nice to meet you, %s\n", "Tao");

	return 0;
}
```