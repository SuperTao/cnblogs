使用Cisco无线控制器管理AP，配置Radius服务器，用于企业加密wifi的认证。

结合上一篇文档进行操作：

http://www.cnblogs.com/helloworldtoyou/p/8033072.html

参考文档：

https://www.cisco.com/c/dam/global/zh_cn/products/products_netsol/wireless/pdf/wireless_lan_radius_1219.pdf

https://www.cisco.com/c/zh_cn/support/docs/wireless-mobility/wlan-security/69730-eap-auth-wlc.html

从无线控制器 GUI 界面单击“SECURITY”。从左侧的菜单中单击 RADIUS > Authentication。RADIUS 认证服务器页面出现，新建添加一个新的 RADIUS 服务器。点击右上角的New按钮，在 RADIUS Authentication Servers > New 页面，输入 RADIUS 服务器的具体参数。下面是一个例子。
 
![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213152902582-2107560250.png)

Server IP Address，就是搭建了RADIUS服务器的电脑的IP地址。
Shared Secret这个密码要和RADIUS安装时的文件/etc/hostapd.radius_clients中的一样，都设置成“test”。
设置成功，显示如下。
 
![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213152915019-145613227.png)

新建AP，如下图所示，WLAN中，选择Create  New，选择go。
 
![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213152925988-171232421.png)

设置Profile Name和SSID。

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213152937754-1736318627.png)

点击Apply，界面按下面设置。

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213153000176-933515848.png)

Security>Layer 2设置如下图，选择WPA+WPA2。

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213153019144-929176003.png)
 
AAA Servers，Server 1选择刚才设置的RADIUS服务器。如图所示。

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213153029582-273029796.png)

高级选项Advanced，需要指定DHCP服务器的IP地址，否者连接的设备获取不到IP，提示连接不上。

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213153044472-515572581.png)
 
设置之后的结果，出现了一个”wifi_test”的WLAN。需要Enabled这个WLAN。

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213153059254-82934018.png)


Tony Liu

2017-12-13