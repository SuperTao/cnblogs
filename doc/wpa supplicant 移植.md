最近移植wifi,WIFI芯片使用rtl8723。在文件系统生成了设备节点。需要移植工具进行测试：

* iwconfig:没有密码的或者wep加密的wifi,使用iwconfig就已经够用。

* wpa_supplicant: 对于wpa/wpa2加密的就需要使用wpa_supplicant进行。在android上底层也是使用wpa_supplicant进行wifi的连接测试等。

移植iwconfig很快就成功，连接无密码的wifi也能成功。

而wpa_supplicant，无论是连接wpa/wpa2还是没有密码的WIFI都是连上了之后就马上断开,然后又从新连接。

网上关于wpa_supplicant移植的很多，也看了很多的资料，收集了起来做个总结。

安装wpa_supplicant之前需要移植

* libnl

* openssl

* TomMath

TomMath是否必须我没有进一步验证，看到有文档要安装，所以就一并安装了。libnl和openssl是必须的。

#### 参考链接

http://www.linuxfromscratch.org/blfs/view/svn/basicnet/wpa_supplicant.html

http://wiki.beyondlogic.org/index.php?title=Cross_Compiling_iw_wpa_supplicant_hostapd_rfkill_for_ARM

http://blog.chinaunix.net/uid-29165999-id-4447034.html

http://blog.chinaunix.net/uid-26921272-id-3416832.html

http://www.arm9home.net/simple/index.php?t20393.html

http://www.linuxidc.com/Linux/2011-10/45202.htm

http://blog.csdn.net/hinyunsin/article/details/6029403

http://blog.csdn.net/dropping_1979/article/details/9621593

#### libnl移植

```
1、设置目录用于安装libnl。

    cd libnl-3.2.25

    mkdir install

2、配置

    ./configure --prefix=/home/Qt/tool/libnl-3.2.25/install CC=arm-linux-gcc LD=arm-linux-ld --enable-shared --enable-static --host=arm-linux

3、编译

    make

4、安装

    make install

5、将install/lib/下的libnl-3.so.200,libnl-genl-3.so.200放到开发板的/lib目录。
```

#### openssl 移植

```
1、设置目录用于安装openssl。

    cd openssl-1.0.1e/

    mkdir install

2、更改Makefile的内容。

    CC= arm-linux-gcc
    AR= arm-linux-ar $(ARFLAGS) r
    RANLIB= arm-linux-ranlib
    # 要是用绝对路径，否者会出错
    INSTALLTOP=/home/Qt/tool/openssl-1.0.1e/install
    OPENSSLDIR=/home/Qt/tool/openssl-1.0.1e/install

3、编译

    make 

4、安装

    make install

5、将install/lib/下的 libcrypto.a和libssl.a 添加到开发板的/lib目录。
```

#### tommath 移植

```
cd libtommath-1.0

make CC=arm-linux-gcc

产生的libtommath.a放到开发板的/lib目录。
```

#### wpa_supplicant 移植

```
1、生成文件配置

cd wpa_supplicant-2.0/wpa_supplicant

cp defconfig .config


2. 更改.config内容

CFLAGS += -I/home/Qt/tool/openssl-1.0.1e/install/include/
LIBS += -L/home/Qt/tool/openssl-1.0.1e/install/lib/
CC=arm-linux-gcc -L/home/Qt/tool/openssl-1.0.1e/install/lib/

CONFIG_DRIVER_NL80211=y
CONFIG_LIBNL32=y
CFLAGS += -I/home/Qt/tool/libnl-3.2.25/install/include/libnl3
LIBS += -L/home/Qt/tool/libnl-3.2.25/install/lib

CONFIG_DRIVER_WEXT=y    
CONFIG_IEEE8021X_EAPOL=y    

CONFIG_EAP_FAST=y        

CONFIG_WPS=y    
CONFIG_EAP_WPS=y 

CONFIG_TLS=openssl        
CONFIG_TLS=internal        

CONFIG_INTERNAL_LIBTOMMATH=y
ifndef CONFIG_INTERNAL_LIBTOMMATH
LTM_PATH=/home/Qt/tool/libtommath-1.0     
CFLAGS += -I$(LTM_PATH)
LIBS += -L$(LTM_PATH)
LIBS_p += -L$(LTM_PATH)
endif

3. 编译

    make

4. 将生成的wpa_supplicant等工具添加到/bin目录。

cp wpa_supplicant/examples/wpa-psk-tkip.conf   rootfs/etc/wpa_supplicant.conf

```

连接的时候更新开发板的/etc/wpa_supplicant.conf中的内容。

运行wpa_supplicant -B -Dnl80211 -i wlan0 -c /etc/wpa_supplicant.conf

#### 错误总结

1.一开始编译wpa_supplicant编译不过。

错误：

fatal error: netlink/genl/genl.h: No such file or directory 

安装了libnl也不行，一开始没有指定libnl的库。 

网上有说可以将 注释 CONFIG_DRIVER_NL80211=y

注释之后，编译。但是又遇到了2的问题。

2.在开发板运行,使用-D指定nl80211设备的时候出错。

wpa_supplicant -B -Dnl80211 -i wlan0 -c /etc/wpa_supplicant.conf

所以就只能使用 -D wext或者 -Dwired 

wpa_supplicant -B -Dwext -i wlan0 -c /etc/wpa_supplicant.conf 

wpa_supplicant -B -Dwired -i wlan0 -c /etc/wpa_supplicant.conf

也许使用wext/wired对于某些设备可行，但是对与我的设备。出现错误，wifi连上几秒钟又断开，然后又重新连接。

```
root@freescale ~$ RTL871X: set bssid:8c:be:be:01:54:d2
RTL871X: set ssid [chinanet] fw_state=0x00000088
RTL871X: start auth
RTL871X: auth success, start assoc
RTL871X: assoc success
RTL871X: OnDisassoc(wlan0) reason=7, ta=8c:be:be:01:54:d2
RTL871X: OnDeAuth(wlan0) reason=7, ta=8c:be:be:01:54:d2, ignore=0
RTL871X: set bssid:00:00:00:00:00:00
RTL871X: set bssid:b0:d5:9d:81:eb:92
RTL871X: set ssid [Aplex_CCC] fw_state=0x00000088
RTL871X: start auth
RTL871X: auth success, start assoc
RTL871X: assoc success
RTL871X: rtw_aes_decrypt(wlan0) no_gkey_bc_cnt:0, no_gkey_mc_cnt:2
RTL871X: rtw_aes_decrypt(wlan0) no_gkey_bc_cnt:0, no_gkey_mc_cnt:1
RTL871X: OnDisassoc(wlan0) reason=7, ta=b0:d5:9d:81:eb:92
RTL871X: OnDeAuth(wlan0) reason=7, ta=b0:d5:9d:81:eb:92, ignore=0
RTL871X: set bssid:00:00:00:00:00:00
RTL871X: set bssid:8c:be:be:01:54:d2
```

3.所以从新编译wpa_supplicant，在.config添加CONFIG_DRIVER_NL80211=y，并指定nl的路径。详见本文的libnl移植。

----

Tony Liu

2016-12-8