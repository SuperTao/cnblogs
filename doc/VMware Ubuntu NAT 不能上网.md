在VMware中配置NAT，控制面板->网络和Internet->网络连接，设置对应的VMware网卡为DHCP。

ubuntu虚拟机中配置网卡为DHCP。获取不到ip。

参考链接：

http://blog.csdn.net/xueyushenzhou/article/details/50460183

需要打开对应的服务：

控制面板->-管理工具--->服务 ，查找 VMware DHCP Service 和VMware NAT Service

保证这两个服务已经启动。并且将启动类型改为自动，否则下次开机又需要手动进行启动。

![](http://images2015.cnblogs.com/blog/745188/201705/745188-20170520100015635-741677103.png)

Tony Liu

2017-5-19, Shenzhen