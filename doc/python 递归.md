学习python，正好用一个例子练习一下递归。

##### 参考文档：

　　http://www.runoob.com/python/python-exercise-example18.html

题目：求s=a+aa+aaa+aaaa+aa...a的值，其中a是一个数字。例如2+22+222+2222+22222(此时共有5个数相加)，几个数相加有键盘控制。

##### 递归方法

```
#!/usr/bin/env python

# 获取单个数字
def get_num(num, bit):
	if bit == 1:
		return num
	return get_num(num, bit-1) * 10 + num

# 将所有的相加
def add_num(num, count):
	if count == 1:
		return num
	return add_num(num, count-1) + get_num(num, count)

if __name__ == '__main__':
	num = int(raw_input('num:'))
	count = int(raw_input('count:'))
	print add_num(num, count)
```

##### 网站的参考程序

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

Tn = 0
Sn = []
n = int(raw_input('n = :\n'))
a = int(raw_input('a = :\n'))
# 计算出每一个数字，添加到列表中
for count in range(n):
    Tn = Tn + a
    a = a * 10
    Sn.append(Tn)
    print Tn

# 计算列表中每个元素的和
Sn = reduce(lambda x,y : x + y,Sn)
print Sn
```

Tony Liu

2016-9-22, Shenzhen