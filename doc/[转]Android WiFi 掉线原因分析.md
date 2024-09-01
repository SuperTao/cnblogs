看到一个比较详细的分析wifi断开的文章。收藏一下。

原文：
	
http://blog.csdn.net/chi_wy/article/details/50963279

```
原因１
．从Log分析来看，这个是由于Dhcp request fail 导致最终disconnect .
Log 分析如下： 
16:53:31.659 958 6525 D NetUtils: dhcp_do_request failed : wlan0 (renew)
08-26 16:53:31.659 958 6525 E DhcpStateMachine: DHCPV4 failed on wlan0: Timed out waiting for DHCP Renew to finish
16:53:41.568 4013 4013 D wpa_supplicant: wlan0: Deauthentication notification
16:53:41.568 4013 4013 D wpa_supplicant: Deauthentication frame IE(s) - hexdump(len=0): [NULL]
16:53:41.569 4013 4013 D wpa_supplicant: wlan0: State: COMPLETED -> DISCONNECTED
需厂商更新一个firmware ，用Attach中的firmware 重新测试。 


原因2.
看到fail的地方都是在四次握手的时候发生的失败 
03-06 18:23:06.585650 1024 1024 D wpa_supplicant: wlan0: State: 4WAY_HANDSHAKE -> DISCONNECTED
03-06 18:23:06.586188 749 1247 D WifiHW : [1] get event: IFNAME=wlan0 <3>WPA: 4-Way Handshake failed - pre-shared key may be incorrect
03-06 18:23:06.586198 749 1247 D WifiHW : [2] get event: IFNAME=wlan0 WPA: 4-Way Handshake failed - pre-shared key may be incorrect
失败应该是客户密码输入错误。如果手机还是一直连上不，可以把手机里的wpa_supplicant.conf文件拉出来，看下servicecenter这个AP的密码是否和预期的一样

原因３
经常reconnect是因为连接的AP信号太弱这时又没有发现有信号更好的AP，导致断线，当此AP信号稍好时又一次scan到此AP，这样才叫reconnect；而roaming是指连接的AP信号太弱这时发现有信号更好的AP，连接到信号更好的AP，这样叫roaming。从log_18_new这份log看到不间断的出现：
请注意： 
像12-09 22:43:18.203 922 922 D wpa_supplicant: wlan0: State: ASSOCIATED -> COMPLETED
12-09 23:13:28.204 922 922 D wpa_supplicant: wlan0: State: COMPLETED -> DISCONNECTED
12-09 23:13:28.305 922 922 D wpa_supplicant: wlan0: State: DISCONNECTED -> SCANNING
12-09 23:13:31.015 922 922 D wpa_supplicant: wlan0: State: SCANNING -> ASSOCIATING
12-09 23:13:31.092 922 922 D wpa_supplicant: wlan0: State: ASSOCIATING -> ASSOCIATED
12-09 23:13:31.093 922 922 D wpa_supplicant: wlan0: State: ASSOCIATED -> COMPLETED
这种就是发生了disconnect，然后reconnect成功，而我们在disconnect时间点没有看到其他引起断线的trace，那就说明是firmware上报的断线请求，可以看到：
12-09 20:38:49.199 922 922 D wpa_supplicant: wlan0: Event INTERFACE_STATUS (5) received
12-09 20:38:49.199 922 922 D wpa_supplicant: nl80211: Event message available
12-09 20:38:49.199 922 922 D wpa_supplicant: wlan0: Event DISASSOC (1) received
12-09 20:38:49.199 922 922 D wpa_supplicant: wlan0: Disassociation notification
而firmware上报断线的原因只有一种，就是它认为RSSI太弱了 
像12-10 04:37:40.633 922 922 D wpa_supplicant: wlan0: State: ASSOCIATED -> COMPLETED
12-10 05:09:43.609 922 922 D wpa_supplicant: wlan0: State: COMPLETED -> ASSOCIATED
12-10 05:09:43.609 922 922 D wpa_supplicant: wlan0: State: ASSOCIATED -> COMPLETED
这种就是发生了roaming，发生roaming的前提就是发现当前AP信号较弱，同时scan到了信号更好的AP．

原因４：
从log来看是AP要求设备断线的： 
09-06 10:48:56.919 <3>[ 1555.321037] (1)[3319:tx_thread][wlan] Rx Deauth frame from BSSID=[aa:63:df:5c:db:c5].
09-06 10:48:56.919 <3>[ 1555.321093] (1)[3319:tx_thread][wlan] Reason code = 7

原因５：
log中看到断线原因如下，是AP更改了security，导致断线 
04-20 11:32:37.439 <4>[ 3273.328514]<2> (1)[4122:tx_thread][wlan] rsnCheckSecurityModeChanged: (RSN INFO) security change, WPA2->WEP
04-20 11:32:37.439 <4>[ 3273.328530]<2> (1)[4122:tx_thread][wlan] scanProcessBeaconAndProbeResp: (SCN INFO) Beacon security mode change detected
04-20 11:32:37.439 <4>[ 3273.328544]<2> (1)[4122:tx_thread][wlan] aisFsmStateAbort_NORMAL_TR: (INIT INFO) aisFsmStateAbort_NORMAL_TR
04-20 11:32:37.439 <4>[ 3273.328559]<2> (1)[4122:tx_thread][wlan] aisFsmDisconnect: (INIT INFO) aisFsmDisconnect

有两次断开连接的情况，具体如下 
28191 04-09 18:39:16.715 3889 3889 D wpa_supplicant: nl80211: Associated on 2422 MHz
28192 04-09 18:39:16.715 3889 3889 D wpa_supplicant: nl80211: Associated with 10:0e:0e:20:61:15
发送roaming，断开与10:0e:0e:20:61:15的连接，重新连接另一个AP:10:0e:0e:20:5e:7d
33517 04-09 18:39:43.054 3889 3889 D wpa_supplicant: nl80211: Drv Event 47 (NL80211_CMD_ROAM) received for wlan0
33518 04-09 18:39:43.054 3889 3889 D wpa_supplicant: nl80211: Roam event
33519 04-09 18:39:43.054 3889 3889 D wpa_supplicant: nl80211: Associated on 2472 MHz
33520 04-09 18:39:43.054 3889 3889 D wpa_supplicant: nl80211: Associated with 10:0e:0e:20:5e:7d
33521 04-09 18:39:43.054 3889 3889 D wpa_supplicant: nl80211: Operating frequency for the associated BSS from scan results: 2472 MHz
33543 04-09 18:39:43.055 3889 3889 D wpa_supplicant: Operating frequency changed from 2422 to 2472 MHz
33544 04-09 18:39:43.055 3889 3889 D wpa_supplicant: nl80211: Associated on 2472 MHz
33545 04-09 18:39:43.055 3889 3889 D wpa_supplicant: nl80211: Associated with 10:0e:0e:20:5e:7d
33546 04-09 18:39:43.055 3889 3889 D wpa_supplicant: nl80211: Received scan results (1 BSSes)
33549 04-09 18:39:43.055 3889 3889 D wpa_supplicant: wlan0: State: COMPLETED -> ASSOCIATED
上层send disable network command,10:0e:0e:20:5e:7d断开连接
39804 04-09 18:40:07.283 3889 3889 D wpa_supplicant: wlan0: Control interface command 'DISABLE_NETWORK 1'
39805 04-09 18:40:07.283 3889 3889 D wpa_supplicant: CTRL_IFACE: DISABLE_NETWORK id=1
39806 04-09 18:40:07.283 3889 3889 D wpa_supplicant: wlan0: Request to deauthenticate - bssid=10:0e:0e:20:5e:7d pending_bssid=00:00:00:00:00:00 reason=3 state=COMPLETED
39808 04-09 18:40:07.283 3889 3889 D wpa_supplicant: wpa_driver_nl80211_disconnect(reason_code=3)
39817 04-09 18:40:07.298 3889 3889 D wpa_supplicant: wlan0: Event DEAUTH (12) received
39832 04-09 18:40:07.299 3889 3889 D wpa_supplicant: addr=10:0e:0e:20:5e:7d
39833 04-09 18:40:07.299 3889 3889 D wpa_supplicant: wlan0: State: COMPLETED -> DISCONNECTED

log2:
此问题从目前的Log分析来看，是由于这个Ap发了一个Deatuh的 Frame ，导致disconnect ，因此是Ap的行为。重新换Ap测试下。
Log分析如下： 
mainlog : 
08-24 11:24:15.410 1097 1097 D wpa_supplicant: nl80211: Drv Event 48 (NL80211_CMD_DISCONNECT) received for wlan0
08-24 11:24:15.410 1097 1097 D wpa_supplicant: nl80211: Disconnect event
08-24 11:24:15.410 1097 1097 D wpa_supplicant: wlan0: Event DEAUTH (12) received
08-24 11:24:15.410 1097 1097 D wpa_supplicant: wlan0: Deauthentication notification
08-24 11:24:15.410 1097 1097 D wpa_supplicant: wlan0: * reason 101
08-24 11:24:15.410 1097 1097 D wpa_supplicant: Deauthentication frame IE(s) - hexdump(len=0): [NULL]
08-24 11:24:15.410 1097 1097 I wpa_supplicant: wlan0: CTRL-EVENT-DISCONNECTED bssid=42:0e:0e:20:6a:d2 reason=101
08-24 11:24:15.410 1097 1097 D wpa_supplicant: wlan0: Disconnect event - remove keys
08-24 11:24:15.411 715 1217 D WifiMonitor: Event [IFNAME=wlan0 CTRL-EVENT-DISCONNECTED bssid=42:0e:0e:20:6a:d2 reason=101]
08-24 11:24:15.412 715 1217 E WifiMonitor: WifiMonitor notify network disconnect: null reason=0
08-24 11:24:15.420 1097 1097 D wpa_supplicant: wlan0: State: COMPLETED -> DISCONNECTED
从上面可以看出是由于收到了；discon所以导致断线！ 
而从kernellog 对应来分析： 
08-24 11:24:15.380 <4>[ 8153.620603]<0>-(0)[936:tx_thread][wlan] nicTxReleaseResource: (TX STATE) Release: TC4 count 1, Free=4; TC5 count 0, Free=2
08-24 11:24:15.395 <4>[ 8153.635513]<0> (0)[936:tx_thread][wlan] saaFsmRunEventRxDeauth: (SAA INFO) Rx Deauth frame from BSSID=[42:0e:0e:20:6a:d2].
08-24 11:24:15.395 <4>[ 8153.635553]<0> (0)[936:tx_thread][wlan] saaFsmRunEventRxDeauth: (SAA INFO) Deauth reason = 101
08-24 11:24:15.395 <4>[ 8153.635912]<0> (0)[936:tx_thread][wlan] nicMediaStateChange: (INIT TRACE) DisByMC
08-24 11:24:15.395 <4>[ 8153.635970]<0> (0)[936:tx_thread][wlan] kalIndicateStatusAndComplete: (INIT INFO) [wifi] wlan0 netif_carrier_off 2
08-24 11:24:15.399 <4>[ 8153.639801]<0> (0)[936:tx_thread][wlan] kalIndicateStatusAndComplete: (INIT INFO) [wifi] wlan0 cfg80211_disconnected
08-24 11:24:15.399 <6>[ 8153.639876]<0> (0)[12175:kworker/0:1][mtk_net][sched]dev_deactivate dev = wlan0
08-24 11:24:15.399 <4>[ 8153.640014]<0> (0)[936:tx_thread][wlan] aisFsmSteps: (AIS STATE) [13] TRANSITION: [10] -> [0]
从上面可以看出是：手机收到了Rx Deauth frame from BSSID=[42:0e:0e:20:6a:d2]. 从而导致断线。
因此此问题从上面Log具体来分析是Ap的行为。


原因６：
从kernel log可以看到，wlan driver在数据传输测试中，因为系统memory太低，alloc skb buffer一直失败，482~517一直印出下面的backtrace：
<4>[ 482.458525] (2)[5:kworker/u:0]CPU 1: hi: dump_backtrace+0x0/0 77
<4>[ 482.458539] (3)[2576:tx_thread][<c00122a8>] (dump_backtrace+0x0/0x10c) from [<c06e004c>] (dump_stack+0x18/0x1c)
<4>[ 482.458550] (3)[2576:tx_thread] r6:ddf5c000 r5:00000000 r4:00000020CP3:c0996244 r3:c0996244
<4>[ 482.458577] (2)[5:kworker/u:0]CPU 3: hi: 186, btch: 31 usd: 59
<4>[ 482.458590] (3)[2576:tx_thread][<c06e0034>] (dump_stack+0x0/0x1c) from [<c00d24e0>] (warn_alloc_failed+0xd0/0x114)
<4>[ 482.458611] (3)[2576:tx_thread][<c00d2410>] (warn_alloc_failed+0x0/0x114) from [<c00d4390>] (__alloc_pages_nodemask+0x3c4/0x828)
<4>[ 482.458627] (2)[5:kworker/u:0]active_anon:22059 inactive_anon:22130 isolated_anon:0
<4>[ 482.458634] (2)[5:kworker/u:0] active_file:11548 inactive_file:31936 isolated_file:0
<4>[ 482.458640] (2)[5:kworker/u:0] unevictable:1190 dirty:14552 writeback:2556 unstable:0
<4>[ 482.458646] (2)[5:kworker/u:0] free:255 slab_reclaimable:4501 slab_unreclaimable:6252
<4>[ 482.458653] (2)[5:kworker/u:0] mapped:5965 shmem:127 pagetables:2348 bounce:0
<4>[ 482.458671] (3)[2576:tx_thread] r3:00000000Normal free:1020kB min:2804kB low:8304kB high:9004kB active_anon:88236kB inactive_anon:88520kB active_file:46192kB inactive_file:127744kB unevictable:4760kB isolated(anon):0kB isolated(file):0kB present:492140kB mlocked:3004kB dirty:58208kB writeback:10224kB mapped:23860kB shmem:508kB slab_reclaimable:18004kB slab_unreclaimable:25008kB kernel_stack:6384kB pagetables:9392kB unstable:0kB bounce:0kB writeback_tmp:0kB pages_scanned:61 all_unreclaimable? no 
<4>[ 482.458708] (2)[5:kworker/u:0]lowmem_reserve[]: 0 0 0
<4>[ 482.458721] (2)[5:kworker/u:0]Normal: 252:00000000 r2:00000000
<4>[ 482.458735] (3)[2576:tx_thread] r7:c09ad010 r6:00000000 r5:000000300*8kB 0*16kB 0*32kB 0*64kB 0*128kB 0*256kB 0*512kB 0*1024kB 0*2048kB 0*4096kB = 1020kB
<4>[ 482.458774] (2)[5:kworker/u:0]44377 total pagecache pages
<4>[ 482.458782] (2)[5:kworker/u:0]249 pages in swap cache
<4>[ 482.458792] (2)[5:kworker/u:0]Swa:00000000 r4:00000000
<4>[ 482.458804] (2)[5:kworker/u:0]Free swap = 375288kB
<4>[ 482.458816] (3)[2576:tx_thread][<c00d3fcc>] (__alloc_pages_nodemask+0x0/0x828) from [<c00feb88>] (new_slab+0x1f0/0x21c)
<4>[ 482.458830] (2)[5:kworker/u:0]Total swap = 393212kB
<4>[ 482.458844] (3)[2576:tx_thread][<c00fe998>] (new_slab+0x0/0x21c) from [<c06e2ac8>] (__slab_alloc.isra.51.constprop.55+0x514/0x534)
<4>[ 482.458861] (2)[5:kworker/u:0]SLUB: Unable to allocate memory on node -1 (gfp=0x20)
<4>[ 482.458875] (3)[2576:tx_thread][<c06e25b4>] (__slab_alloc.isra.51.constprop.55+0x0/0x534) from [<c010005c>] (__kmalloc_track_caller+0x10c/0x160)
<4>[ 482.458899] (3)[2576:tx_thread][<c00fff50>] (__kmalloc_track_caller+0x0/0x160) from [<c059fa30>] (__alloc_skb+0x58/0xf4)
<4>[ 482.458910] (2)[5:kworker/u:0] cache: kmalloc-2048, object size: 2048, buffer size: 2048, default order: 0, min order: 0
<4>[ 482.458922] (2)[5:kworker/u:0] node 0: slabs: 0, objs: 0, free: 0
<4>[ 482.458946] (3)[2576:tx_thread][<c059f9d8>] (__alloc_skb+0x0/0xf4) from [<c059faec>] (dev_alloc_skb+0x20/0x44)
<4>[ 482.458973] (3)[2576:tx_thread][<c059facc>] (dev_alloc_skb+0x0/0x44) from [<c042b4a8>] (kalPacketAlloc+0x18/0x28)
<4>[ 482.459005] (3)[2576:tx_thread]kworker/u:0: page allocation failure: order:0, mode:0x20
<4>[ 482.459014] (2)[5:kworker/u:0]kworker/u:0: page allocation failure: order:0, mode:0x20
<4>[ 482.459023] (2)[5:kworker/u:0]Backtrace:
<4>[ 482.459034] (3)[2576:tx_thread] r4:e226a4c0[<c00122a8>] (dump_backtrace+0x0/0x10c) from [<c06e004c>] (dump_stack+0x18/0x1c)
<4>[ 482.459054] (2)[5:kworker/u:0] r6:ddc32000 r3:000042f0
<4>[ 482.459067] (2)[5:kworker/u:0] r5:00000000[<c041a470>] (nicRxSetupRFB+0x0/0xf4) from [<c03f93b0>] (wlanReturnPacket+0xc4/0x26c)
<4>[ 482.459087] (3)[2576:tx_thread] r5:000042f0 r4:e226a4c0
<4>[ 482.459101] (2)[5:kworker/u:0] r4:00000020[<c03f92ec>] (wlanReturnPacket+0x0/0x26c) from [<c042b650>] (kalRxIndicatePkts+0x168/0x298)
<4>[ 482.459120] (3)[2576:tx_thread] r7:d72b0000 r6:e2212b7c r5:dd3602e0 r3:00000000 r4:ce9d9e40
<4>[ 482.459143] (3)[2576:tx_thread] 
<4>[ 482.459154] (2)[5:kworker/u:0][<c06e0034>] (dump_stack+0x0/0x1c) from [<c00d24e0>] (warn_alloc_failed+0xd0/0x114)
<4>[ 482.459175] (2)[5:kworker/u:0][<c00d2410>] (warn_alloc_failed+0x0/0x114) from [<c00d4390>] (__alloc_pages_nodemask+0x3c4/0x828)
<4>[ 482.459191] (3)[2576:tx_thread][<c042b4e8>] (kalRxIndicatePkts+0x0/0x298) from [<c041c2e8>] (nicRxProcessRFBs+0x1e8/0x294)
<4>[ 482.459206] (2)[5:kworker/u:0] r3:00000000[<c041c100>] (nicRxProcessRFBs+0x0/0x294) from [<c041c79c>] (nicProcessRxInterrupt+0x24/0x5c)
<4>[ 482.459229] (2)[5:kworker/u:0] r2:00000000[<c041c778>] (nicProcessRxInterrupt+0x0/0x5c) from [<c0411d5c>] (nicProcessIST_impl+0x64/0x1b4)
最后FreeSwRfb在518时耗尽，这个时间点正好是tcpdump显示PC端没有回应的开始： 
<4>[ 518.094029] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094055] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094281] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094329] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094379] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094395] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094442] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094601] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094648] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094697] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094714] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
<4>[ 518.094762] (1)[2576:tx_thread][wlan] qmHandleRxPackets: (QM TRACE) Mark NULL the Packet for less Free Sw Rfb
```