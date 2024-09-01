WFA认证需要启动sigma_dut，记录记录一下启动过程。

Android O平台命令如下

```
adb shell svc wifi disable
adb shell rmmod wlan 
adb shell insmod /vendor/lib/modules/wlan.ko
注意，运行wpa_supplicant之后，不要将这个窗口关闭，也不要用control + c 终止程序。
adb shell /vendor/bin/hw/wpa_supplicant -ip2p0 -Dnl80211 -c /data/misc/wifi/p2p_supplicant.conf -N -iwlan0 -Dnl80211 -c /data/misc/wifi/wpa_supplicant.conf -dddd -e /data/misc/wifi/entropy.bin -puse_p2p_group_interface=1 -O /data/misc/wifi/sockets
另外再开一个窗口运行下面的命令。
adb shell /system/bin/sigma_dut -d -w /data/misc/wifi/sockets/ -p 9000 -M wlan0 -S wlan0 -c WCN -C /data/cert/
```
android O上运行完wpa_supplicant才可以看到wlan0的接口。如果终止了wpa_supplicant或者手动启动wlan0都是无效的。

android N 安装完wlan.ko模块就有wlan0的接口。

sigma_dut可以自己编译，最好找高通要官方的。

将国家码设置成美国方法：

`adb shell wpa_cli -iwlan0 DRIVER COUNTRY US`



Tony Liu

2018-5-16