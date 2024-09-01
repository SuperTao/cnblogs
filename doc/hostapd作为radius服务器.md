使用hostapd作为radius服务器，用于企业wifi加密认证。

参考链接：

http://www.cnblogs.com/claruarius/p/5902141.html

去网上下载hostapd 源码，本文使用的是hostapd-2.4。操作系统ubuntu16.04。

```
1,tar -xvzf hostapd-2.4.tar.gz
2,cd hostapd-2.4/hostapd
3,cp defconfig .config
4,open options in .config
#CONFIG_DRIVER_NL80211=y
CONFIG_DRIVER_NONE=y
CONFIG_EAP_SIM=y
CONFIG_EAP_AKA=y
CONFIG_EAP_AKA_PRIME=y
CONFIG_EAP_PAX=y
CONFIG_EAP_PSK=y
CONFIG_EAP_PWD=y
CONFIG_EAP_SAKE=y
CONFIG_EAP_GPSK=y
CONFIG_EAP_GPSK_SHA256=y
CONFIG_EAP_FAST=y
CONFIG_RADIUS_SERVER=y

5,make
6,sudo make install
**************
if use debian 9, just run "apt install hostapd"
7,modify hostapd.conf
Interface=eno1
driver=none
eap_server=1
eap_user_file=/etc/hostapd.eap_user
ca_cert=/usr/local/etc/raddb/certs/cas.pem
server_cert=/usr/local/etc/raddb/certs/server.pem
private_key=/usr/local/etc/raddb/certs/server.key
private_key_passwd=whatever
dh_file=/usr/local/etc/raddb/certs/dh
pac_opaque_encr_key=000102030405060708090a0b0c0d0e0f
eap_fast_a_id=101112131415161718191a1b1c1d1e1f
eap_fast_a_id_info=test server
eap_fast_prov=3
pac_key_lifetime=604800
pac_key_refresh_time=86400
radius_server_clients=/etc/hostapd.radius_clients
radius_server_auth_port=1812
radius_server_acct_port=1813

Interface=eno1，这个需要根据情况进行更改。Ubuntu16.04 中已经不存在eth0，eth1
等接口，有线网卡的名称已经更改为eno1。使用ifconfig 命令可以查看有线网卡的设备名
称。
上面如果没有/usr/local/etc/raddb/certs/目录，需要自己创建这个目录，并把相应的文件
放到对应的目录中。可以使用自己的文件放到对应的目录中,我们是买了WFA的证书。
或者server.pem, server.key, cas.pem可以复用hostapd文件夹下的同名文件，即拷贝过去配置好相对路径就好。

8,sudo cp hostapd.eap_user /etc/
9,modify /etc/hostapd.eap_user
Delete all default user, add:
"testing" FAST,PEAP,TTLS,TLS
"testiing1" FAST,PEAP,TTLS,TLS
"test" FAST,PEAP,TTLS,TLS
"Barney Rubble" FAST,PEAP,TTLS,TLS
"testing" MSCHAPV2,GTC,MD5,TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,TTLSMSCHAPV2
"password" [2]
"test" MSCHAPV2,GTC,MD5,TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,TTLSMSCHAPV2
"test" [2]


这个文件可以使用hostapd_new_install.zip 压缩包中的同名文件。也可以使用hostapd压缩包中默认的文件。

10,sudo cp hostapd.radius_clients /etc/
11,modify /etc/hostapd.radius_clients
Delete all default client, add: (secret is test)
0.0.0.0/0 test
这个“test”是登陆的密码，在设置RADIUS 服务器的时候，share secret 要设置成一
样，后面会提到。
12, generate dh file
openssl dhparam -out dh 1024
mv dh /usr/local/etc/raddb/certs/
13, sudo hostapd -dd hostapd.conf
```
运行结果

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171213151740410-740574740.png)

至此，hostapd作为Radius服务器运行完成。

添加PWD认证方法：

在/etc/hostapd.eap_user,`# Phase 1 User`下面添加

```
"testpwd"    PWD    "test"
```

第一个是ID，第二个是password.


Tony Liu

2017-12-13