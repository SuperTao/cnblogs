记录Android N关联拒绝之后的相关的log.
```
10-26 20:35:35.844  2215  2215 D wpa_supplicant:   * bssid_hint=44:48:c1:d6:57:b0
10-26 20:35:35.844  2215  2215 D wpa_supplicant:   * freq=5240
10-26 20:35:37.905  2215  2215 D wpa_supplicant: wlan0: Event ASSOC_REJECT (13) received
-26 20:35:37.906  2215  2215 I wpa_supplicant: wlan0: CTRL-EVENT-ASSOC-REJECT bssid=44:48:c1:d6:57:b0 status_code=1     //出现关联违反

10-26 20:35:38.999  2215  2215 D wpa_supplicant: wlan0: nl80211: New scan results available
10-26 20:35:39.000  2215  2215 D wpa_supplicant: nl80211: Scan included frequencies: 5805 5240 5745 5200 5180 5220 2437 5785 2462 5765    //优化扫描，只扫描一些特定的信道，方便更加快速的重连

10-26 20:35:39.028  2215  2215 D wpa_supplicant: wlan0: Selecting BSS from priority group 1
10-26 20:35:39.029  2215  2215 D wpa_supplicant: wlan0: No APs found - clear blacklist and try again        // 之前的扫描中，没有找到可以连接的AP，所以清楚黑名单
10-26 20:35:39.029  2215  2215 D wpa_supplicant: Removed BSSID 44:48:c1:d6:57:b0 from blacklist (clear)
10-26 20:35:39.030  2215  2215 D wpa_supplicant: wlan0: No suitable network found     //然而扫描结果中没有目标AP

10-26 20:35:55.793  2215  2215 D wpa_supplicant: wlan0: nl80211: New scan results available   //进行全局扫描
10-26 20:35:55.793  2215  2215 D wpa_supplicant: nl80211: Scan included frequencies: 2412 2417 2422 2427 2432 2437 2442 2447 2452 2457 2462 2467 2472 2484 4920 4940 4960 4980 5040 5060 5080 5180 5200 5220 5240 5260 5280 5300 5320 5500 5520 5540 5560 5580 5600 5620 5640 5660 5680 5700 5745 5765 5785 5805 5825

10-26 20:35:55.842  1749  2081 D WifiConnectivityManager: AllSingleScanListener onResults: start QNS
10-26 20:35:55.843  1749  2081 D WifiQualifiedNetworkSelector:: ==========start qualified Network Selection==========
10-26 20:35:55.843  1749  2081 D WifiQualifiedNetworkSelector:: Saved Network List
10-26 20:35:55.852  1749  2081 D WifiQualifiedNetworkSelector:: After user choice adjust, the final candidate is:"coupang":0 : 04:bd:88:b2:af:40
10-26 20:35:55.852  1749  2081 D WifiQualifiedNetworkSelector:: Roaming from "coupang":0 to "coupang":0        //漫游到目标AP

10-26 20:35:55.874  2215  2215 D wpa_supplicant: wlan0: Control interface command 'SAVE_CONFIG'
10-26 20:35:55.874  2215  2215 D wpa_supplicant: Writing configuration file '/data/misc/wifi/wpa_supplicant.conf.tmp'
10-26 20:35:55.880  2215  2215 D wpa_supplicant: Configuration file '/data/misc/wifi/wpa_supplicant.conf' written successfully     //保存目标AP 配置，

10-26 20:35:55.892  2215  2215 D wpa_supplicant: wlan0:     bss->ssid= 04:bd:88:b2:af:40    ssid->bssid=04:bd:88:b2:af:40 ssid='coupang'
10-26 20:35:55.892  2215  2215 D wpa_supplicant: wlan0:    selected based on RSN IE
10-26 20:35:55.892  2215  2215 D wpa_supplicant: wlan0:    selected BSS 04:bd:88:b2:af:40 ssid='coupang'        //从扫描结果中找到目标AP，

```
LiuTao
2018-11-14