本文介绍WCNSS_qcom_cfg.ini中常用参数的作用。

#### wifi 日志等级

* vosTraceEnableBAP=255
* vosTraceEnableTL=255
* vosTraceEnableWDI=255
* vosTraceEnableHDD=255
* vosTraceEnableSME=255
* vosTraceEnablePE=255
* vosTraceEnablePMC=255
* vosTraceEnableWDA=255
* vosTraceEnableSYS=255
* vosTraceEnableVOSS=255
* vosTraceEnableSAP=255
* vosTraceEnableHDDSAP=255
* 
* wdiTraceEnableDAL=255
* wdiTraceEnableCTL=255
* wdiTraceEnableDAT=255
* wdiTraceEnablePAL=255

```
每一位代表一个等级是否打开，0xFF,表示所有的等级都打开。
00000001 FATAL
00000010 ERROR
00000100 WARN
00001000 INFO
00010000 INFO HIGH
00100000 INFO MED
01000000 INFO LOW
10000000 DEBUG

```

#### 省电模式

* gEnableImps=1

* gEnableBmps=1

```
相关参数:
# Enable IMPS or not
gEnableImps=1
# Enable BMPS or not
gEnableBmps=1

gEnableImps:(Idle mode powersave)
打开wifi，未连接wifi的power save。

gEnableBmps:(Beacon mode powersave)
连接wifi的情况下的power save

使用工具测量电池电流大小。

```

#### Phy Mode (auto, b, g, n, etc)

* gDot11Mode=0

```
# Valid values are 0-9, with 0 = Auto, 4 = 11n, 9 = 11ac
gDot11Mode=0

支持何种协议，可取之范围0-9
```

#### Roaming Parameters

* gNeighborLookupThreshold=65

* RoamRssiDiff=5

* gRoamIntraBand=0

```
* gNeighborLookupThreshold

Roaming RSSI Threshold:		ap的信号达到所设置的值，将会进行扫描周边设备，寻找更好的AP
* RoamRssiDiff
Roaming RSSI Difference:	AP将会进行roaming,当寻找到的ap的信号比连接的AP的信号更好，并且超过这个值

* gRoamIntraBand=1
							
# To enable, set gRoamIntraBand=1 (Roaming within band)
# To disable, set gRoamIntraBand=0 (Roaming across band)
enable across band roaming. 只会在同一个频段内部进行roam，例如2.4G到2.4G内部，或者5G到5G。
							如果没有打开，那么roam的时候，2.4G-5G直接的roam是不允许的。
							但是可以断开再重连。
```
	
#### 802.11d支持

* g11dSupportEnabled=1

```
80_Y0476_2_WCN36X0_ANDROID_WLAN_REGULATORY_AND_COUNTRY_CODE.pdf

802.11d支持，根据路由器的国家码进行选择。
g11dSupportEnabled=1
Wifi 国家码获取途径
    1.DefaultCountryTablefield in WCNSS_qcom_wlan_nv.bin-read during driver initialization
    nv中默认有设置国家码
    2.gStaCountryCodeparameter in WCNSS_qcom_cfg.ini –read during driver initialization to replace default country code in WCNSS_qcom_wlan_nv.bin
    配置文件gStaCountryCodeparameter设置国家码，用于覆盖nv中的国家码
    3.Country IE from AP defined by 802.11d –information given by AP
    使能802.11d功能，通过AP来获取国家码，g11dSupportEnabled用来打开这个功能。
    4.“iw reg set” command –set from userspaceapplication over cfg80211 interface
    用户空间通过命令设置国家码
    5.Private IOCTL with “COUNTRY” command –set from userspaceapplication over wextinterface
    用户空间通过ioctl设置国家码,例如wpa_cli -iwlan DRIVER COUNTRY US.

gCountryCodePriority设置国家码获取的优先级
        1 –Country Code information from userspacecommands takes priority
        userspacecommands > 802.11d > gStaCountryCodein WCNSS_qcom_cfg.ini > DefaultCountryTablein WCNSS_qcom_wlan_nv.bin
默认是0：
        0 –Country Code information from 802.11d takes priority
        802.11d > userspacecommands > gStaCountryCodein WCNSS_qcom_cfg.ini > DefaultCountryTablein WCNSS_qcom_wlan_nv.bin
    
通过AP获取国家码：
    g11dSupportEnabled=1使能，然后如果周围有多个AP，包含不同的国家码，根据接收到的Beacon帧(被动扫描)，进行投票，设置成票数最多的国家码。
        但是我查看的话，好多情况下，只是更具了解的AP来设置国家码。  
        gEnableBypass11d=1，会进行主动扫描，获取国家码，这样速率会快些。

通过SIM卡获取国家码是最可靠的途径。

```

记录一下与国家码有关的网址，方便查找：
国家地区代码：

https://zh.wikipedia.org/wiki/%E5%9C%8B%E5%AE%B6%E5%9C%B0%E5%8D%80%E4%BB%A3%E7%A2%BC

wifi信道列表

https://zh.wikipedia.org/wiki/WLAN%E4%BF%A1%E9%81%93%E5%88%97%E8%A1%A8

世界各个地区WIFI 2.4G及5G信道一览表

http://www.sohu.com/a/143179782_202311

kernel中相关文档：

net/wireless/db.txt

#### beacon loss

* gHeartbeat24=40

```

beacon包的统计，如果超过40(默认值)个没有到，表示AP不在范围内。会有相应的事件产生。

验证方法：准备一台可以方便断电操作的路由器 ，Device先连上AP， 开始抓包 ，然后手动让AP断电
，从抓包的记录上看路由器发送的最后一个beacon和设备开始发prob request的时间差。
一般一个AP的beacon时间间隔是0.1秒，如果设置成40，那就是4s.

粗略验证方法，AP断电，查看设备wifi状态从connected变为saved所需要的时间。

```
#### 信道带宽选择

* BandCapability=0

* gOperatingChannelListEnabled

* gOperatingChannelList

```
#Preferred band (both or 2.4 only or 5 only)
BandCapability=0
0: both
1: 5G
2: 2.4G

# Operating Channel List
# 打开信道选择
gOperatingChannelListEnabled=1
不打开就表示所有信道都支持。打开就根据gOperatingChannelList的内容进行显示。
# 所选择的信道
gOperatingChannelList=6,7,8
所支持的信道。

```

#### WMM Enable/Disable

* WmmIsEnabled=0

```
WmmIsEnabled=0
Wifi Multi Media,wifi多媒体。
WMM is enabled:
1 – Enable, QoS only
2 – Enable, but not QoS
0 – Auto, join any AP
Wi-Fi网络中的多媒体应用要求服务质量(QoS)功能。QoS能使Wi-Fi接入点区分业务优先级，
并优化共享网络资源的方法。如果没有QoS，在不同设备上运行的所有应用传送数据帧的
机会相等，这对于网络浏览器、文件传送或E-mail这类应用的数据业务不成问题，但对于
多媒体应用则不适宜。Internet协议上话音(VoIP)、流视频和交互式游戏对时延增加和
吞吐量下降高度敏感，因此要求QoS。
Wi-Fi联盟把Wi-Fi多媒体(WMM)定义为即将实现的IEEE 802.11e标准的规范概要，并开始
实施WMM合格检验计划，以满足业界对Wi-Fi网络QoS解决方案的需求。
```

#### cisco漫游协议支持

* FastTransitionEnabled=1

```
# CCX Support and fast transition
```


Tao Liu

2018-12-27