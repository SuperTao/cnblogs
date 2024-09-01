开机logo旋转方法。

#### 参考链接

https://www.kernel.org/doc/Documentation/fb/fbcon.txt

https://community.nxp.com/thread/309622


#### uboot

bootargs中添加参数。

```
fbcon=rotate:3

后面的数字可以是0-3
```

Tony Liu

2016-12-4, Shenzhen