qt屏幕旋转的方法

#### 参考链接

http://mikenoodle.blog.163.com/blog/static/11333522010102754154616/

http://blog.csdn.net/qiao_yihan/article/details/16845683

https://community.nxp.com/thread/310542

#### 旋转方法

```
1. 添加环境变量
export QWS_DISPLAY=Transformed:Rot90

2. 运行时添加参数
qtdemo -qws  -display "Transformed:Rot90"

3. 操作/sys/class
echo N > /sys/class/graphics/fb0/rotate
N的值是0-4.
```

Tony Liu

2016-12-3, Shenzhen