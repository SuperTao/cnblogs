内核初始化后，启动init进程/sbin/init，读取/etc/inittab文件进行初始化。

#### 参考链接

http://wenku.baidu.com/view/5a82b5f67c1cfad6195fa76f.html

http://www.linux178.com/linux/inittab.html

http://www.mask.org.tw/old/favorite/linux/inittab.html

http://leejia.blog.51cto.com/4356849/788895

#### inittab格式

inittab中每行的格式：

id:runlevels:action:process

#### id 

标识符，不能重复。

#### runlevels

表示运行级别。共六个级别。如果不填表示所有级别。例如2345，表示在级别是2,3,4,5的时候才会执行。

* 0：表示关机
* 1：表示单用户模式，在这个模式中，用户登录不需要密码，默认网卡驱动是不被加载，一些服务不能用。
* 2：表示多用户模式，NFS服务不开启
* 3，表示命令行模式
* 4，这个模式保留未用
* 5，表示图形用户模式
* 6，表示重启系统

#### action 

表示采取的动作。

* sysinit: 表示系统启动。

* respawn: 表示process结束后会重新启动。

* ctrlaltdel: 表示键盘输入组合件ctrl+alt+del的动作。

* initdefalt: 设置默认的运行级别。

* askfirst: 运行程序之前会先询问,输出“Please press Enter to active this console"

* wait: 等相应的进程完成才继续执行。

##### process

表示要运行的程序。

例如

```
id:3:initdefault:      
# 设置默认的运行级别。

ttymxc0::once:/bin/login root	
# id为ttymxc0表示imx6的串口0，
# levels为空，表示所有的级别都执行。
# once表示只执行一次，如果退出了就不会再次执行。
# /bin/login表示运行的程序，root是传给/bin/login的参数
```

---

Tony Liu

2016-12-11, Shenzhen