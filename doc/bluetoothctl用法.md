```
https://blog.csdn.net/chenjk10/article/details/90317028

https://blog.csdn.net/zhuo_lee_new/article/details/106626680

首先启动bluetoothctl启动

常用命令
	power on/off 打开电源或者关闭
	scan on/off 开始扫描
	pair xx:xx:xx:xx:xx 匹配某个设备
	paired-devices显示已经匹配的设备
	connect xx:xx:xx:xx:xx 连接蓝牙设备

在手机的setting界面可以搜索到设备，但是用nrf connect app搜索不到ble设备。
首先运行
	./bluetoothctl
给芯片上电， 
	power on
使芯片可发现 
	menu advertise
	Name xxxx (需要设置名称)
	Discoverable on
打开广播 
	back
	Advertise on

查看属性值：
显示设备支持的所有属性
	list-attribution 
	
	select-attribution
添加自己的service和characteristic
[bluetoothctl] menu gatt
[bluetoothctl] register-service 0xFFFF # (Choose yes when asked if primary service)
[bluetoothctl] register-characteristic 0xAAAA read       # (Select a value of 1 when prompted)# 输入的值是初始值，每次读取，会加一
[bluetoothctl] register-characteristic 0xBBBB read,write # (Select a value of 0 when prompted)
[bluetoothctl] register-characteristic 0xCCCC read       # (Select a value of 2 when prompted)
[bluetoothctl] register-application # (This commits the services/characteristics and registers the profile)
[bluetoothctl] back
[bluetoothctl] advertise on 


作为主机连接从设备

1. # 进入bluetoothctl命令行
2. bluetoothctl
3. # 进入bluetoothctl命令行后，类似：[bluetooth]#
4. # Controller代表arm板上的蓝牙设备，Device代表蓝牙耳机
5. # 按如下步骤初始化蓝牙设备，并连接蓝牙耳机
6. # 查看帮组
7. help
8. # 
9. power on
10. agent on
11. default-agent 
12. # 启动扫描
13. scan on 
14. # 假如蓝牙耳机地址为：E3:28:E9:24:21:07
15. trust E3:28:E9:24:21:07
16. # 配对
17. pair E3:28:E9:24:21:07
18. # 连接
19. connect E3:28:E9:24:21:07
20. # 查看蓝牙耳机信息
21. info E3:28:E9:24:21:07
22. # 断开连接
23. disconnect E3:28:E9:24:21:07
24. # 不想自动连接上蓝牙耳机，可以删除配对信息
remove E3:28:E9:24:21:07
```