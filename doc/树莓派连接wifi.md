使用树莓派，通过无线网卡连接wifi，再通过远程桌面或者ssh的连接树莓派比较方便，本文记录树莓派wifi如何设置。

##### 参考链接：

 　　http://www.jianshu.com/p/b42e8d3df449

##### 配置脚本

vi /etc/network/interfaces

```
auto lo

iface lo inet loopback
iface eth0 inet dhcp

auto wlan0
allow-hotplug wlan0     # 热插拔
iface wlan0 inet dhcp   # dhcp
wpa-ssid "Tony"         # wifi名称
wpa-psk "password"      # wifi密码
```

Tony Liu

2016-8-28, Shenzhen