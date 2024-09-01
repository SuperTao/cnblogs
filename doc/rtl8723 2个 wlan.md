安装8723bs.ko模块之后，生成了wlan0和wlan1，MAC地址一样。

http://blog.csdn.net/djman007/article/details/46731335

解决方法：

insmod rtl8723.ko ifname=wlan0 if2name=p2p0

wlan0和p2p0共用一个网卡。MAC地址也比较类似。

Tony Liu

2016-12-8, Shenzhen