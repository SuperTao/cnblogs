记录一下Android获取国家码的方式
```
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
wifi信道列表：
https://zh.wikipedia.org/wiki/WLAN%E4%BF%A1%E9%81%93%E5%88%97%E8%A1%A8
世界各个地区WIFI 2.4G及5G信道一览表
http://www.sohu.com/a/143179782_202311
kernel中相关文档：
net/wireless/db.txt

LiuTao
2018-11-15