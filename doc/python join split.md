本文记录python，join和split函数的用法。

#### 参考

　　http://blog.csdn.net/doiido/article/details/43538833

　　http://blog.csdn.net/acdreamers/article/details/8920144

#### join

join() 方法用于将序列中的元素以指定的字符连接生成一个新的字符串。

```
#!/usr/bin/env python

list = ['1','2','3','4']

str = '--'

print str.join(list)

str = '**'

print str.join(list)
'''
1--2--3--4
1**2**3**4
'''
```

os.path.join()：  将多个路径组合后返回

```
import os  

print os.path.join('/hello/','good/boy/','doiido') 
'''
输出：
/hello/good/boy/doiido
'''
```



#### split

```
#!/usr/bin/env python

str = '1--2--3--4'

print str.split()
print str.split('99')
print str.split('--')
'''
输出结果
['1--2--3--4']          # 不存在的字符就直接返回一个新的列表，包含一个圆熟
['1--2--3--4']          # 不存在的字符就直接返回一个新的列表，包含一个圆熟
['1', '2', '3', '4']    # 按照‘'--'分割的结果
'''
```

Tony Liu

2016-9-23， Shenzhen