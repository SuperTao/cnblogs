本文记录python中,map,filter,reduce函数的用法。

参考链接：

　　http://www.python-course.eu/lambda.php

#### map

map(func, seq) 连续将seq中的元素作为参数，调用func。返回经过func函数作用之后，新的序列。

```
#!/usr/bin/env python

list = [1, 2, 3, 4]

print list

ret = map(lambda x: x*2, list)

print ret
'''
输出结果,得到一个新的列表
[1, 2, 3, 4]
[2, 4, 6, 8]
'''
```

#### filter

filter(func, list) 将list元素作为func函数的参数，func将返回一个bool值，true或者false。如果func的结果是true，那么传入的参数作为新序列的一个元素。

```
#!/usr/bin/env python

list = [1, 2, 3, 4, 5]

ret = filter(lambda x: x%2, list)

let = filter(lambda x: x!='a', 'abcde')

print ret
print let

'''
输出结果
[1, 3, 5]
bcde
'''
```

#### reduce

reduce(func, seq) 连续调用将seq中的元素作为参数，连续调用func函数。

reduce(fucn, seq, s) 可以个reduce设定一个初值s

参考链接中的说明。

```
The function reduce(func, seq) continually applies the function func() to the sequence seq. It returns a single value. 

If seq = [ s1, s2, s3, ... , sn ], calling reduce(func, seq) works like this:
At first the first two elements of seq will be applied to func, i.e. func(s1,s2) The list on which reduce() works looks now like this: [ func(s1, s2), s3, ... , sn ]
In the next step func will be applied on the previous result and the third element of the list, i.e. func(func(s1, s2),s3)
The list looks like this now: [ func(func(s1, s2),s3), ... , sn ]
Continue like this until just one element is left and return this element as the result of reduce()
```

举例

```
#!/usr/bin/env python

# test 1
def add(a,b):
	return a + b

list = [1, 2, 3, 4]
print list                   
print reduce(add, list)     
print list
'''
输出结果
[1, 2, 3, 4]
10
[1, 2, 3, 4]
说明运行reduce函数之后，list列表中的元素并没有减少

一开始，list的前2个元素将会作为参数传入add函数，其结果作为新的参数与list第3个元素作为参数再传入add
第1次  add(1,2)
第2次  add(add(1,2),3)
第3次  add(add(add(1,2),3),4)
最后结果 10
'''

# test 2, 计算1-100的值, 输出结果 5050
print reduce(lambda x,y: x+y, range(1,101))

# test 3, 给定一个初值, 输出结果 5070
print reduce(lambda x,y: x+y, range(1,101), 20)
```

Tony Liu

2016-9-22, Shenzhen