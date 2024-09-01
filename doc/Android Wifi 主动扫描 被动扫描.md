介绍主动扫描，被动扫描以及连接的wifi的扫描过程

### 参考文档

《802.11无线网络权威指南》

《80_Y0513_1_QCA_WCN36X0_SOFTWARE_ARCHITECTURE.pdf》(高通文档)

### 被动扫描（passive scanning）
可以节省电池的电力，因为不需要传送任何信号。在被动
扫描中，工作站会在频道表（channel list）所列的各个频道之间不断切换，并静候Beacon 帧
的到来。所收到的任何帧都会被暂存起来，以便取出传送这些帧之BSS 的相关数据。
作被动扫描的过程中，工作站会在频道间不断切换，并且会记录来自所收到之Beacon 信息
的信息。Beacon 在设计上是为了让工作站得知，加入某个基本服务组合（basic service set，
简称BSS）所需要的参数，以便进行通讯。

### 主动扫描（active scanning）
在主动扫描中，工作站扮演比较积极的角色。在每个频道上，工作站
都会岭出Probe Request 帧，请求某个特定网络予以回应。主动扫描系主动试图寻找网络，而不
是听候网络宣告本身的存在。使用主动扫描的工作站将会以如下的程序扫描频道表所列的频道：

1.跳至某个频道，然后等候来讯显示，或者等到ProbeDelay 计时器逾时。如果在这个频道收得到帧，就证明该频道有人使用，因此可以加以探
测。此计时器用来防止某个闲置频道让整个程序停摆；工作站不会一直听候帧到来。

2.利用基本的DCF 访问程序取得介质使用权，然后送出一个Probe Request 帧。

3.至少等候一段最短的频道时间（即MinChannelTime）。

a.如果介质并不忙碌，表示没有网络存在。因此可以跳至下个频道。

b.如果在MinChannelTime 这段期间介质非常忙碌，就继续等候一段时间，直 到
最长的频道时间（即MaxChannelTime），然后处理任何的Probe Response 帧。

当网络收到搜寻其所属之延伸服务组合的Probe Request（探查要求），就会发出Probe
Response（探查回应）帧。为了在舞会中找到朋友，各位或许会绕著舞池大声叫喊对方的名字。
（虽然这并不礼貌，不过如果真想找到朋友，大概没有其他选择。）如果对方听见了，她就会出
声回应，至于其他人根本就不会理你（希望如此）。Probe Request 框的作用类似，不过在Probe
Request 帧当中可以使用broadcast SSID，如此一来，该区所有的802.11 网络都会以Probe
Response 加以回应。

#### probe request两种情况的不同

##### 指定SSID

![](https://img2018.cnblogs.com/blog/745188/201812/745188-20181220155727566-658640503.png)


##### SSID为空(broadcast)

![](https://img2018.cnblogs.com/blog/745188/201812/745188-20181220155743171-539729832.png)


### kernel扫描日志

#### 主动扫描

```
18:49:39.864678  [18:49:39.858008] [00000016B70D009C] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS PE MC Message queue
18:49:39.864704  [18:49:39.858079] [00000016B70D05E4] [VosMC] wlan: [I :PE ] limContinuePostChannelScan: 477:  Mac Addr da:a1:19:53:b9:f8 used in sending ProbeReq number 0, for SSID  on channel: 2
18:49:39.864726  [18:49:39.858146] [00000016B70D0AEA] [VosMC] wlan: [I :VOS] VPKT [779]: [0000000000000000] Packet allocated, type 0[TX_802_11_MGMT]
18:49:39.864748  [18:49:39.858193] [00000016B70D0E76] [VosMC] wlan: [IH:WDA] Tx Mgmt Frame Subtype: 4 alloc(0000000000000000) txBdToken = 0
18:49:39.864770  [18:49:39.858221] [00000016B70D108F] [VosMC] wlan: [I :TL ] TL: using self sta addr to get staidx for spoofed probe req 00:0a:f5:32:b1:50
18:49:39.864792  [18:49:39.858237] [00000016B70D11BE] [VosMC] wlan: [IL:TL ] WLAN TL: fProtMgmtFrame:0
18:49:39.864814  [18:49:39.858256] [00000016B70D132A] [VosMC] wlan: [IL:TL ] WLAN TL: Dump TX meta info: txFlags:2, qosEnabled:0, ac:0, isEapol:0, fdisableFrmXlt:1, frmType:0
18:49:39.864836  [18:49:39.858269] [00000016B70D142E] [VosMC] wlan: [IH:TL ] Serializing WDA TX Start Xmit event
18:49:39.866943  [18:49:39.858720] [00000016B70D360E] [VosTX] wlan: [I :VOS] VosTXThread: Servicing the VOS TL TX Message queue
18:49:39.866992  [18:49:39.858916] [00000016B70D44B8] [VosTX] wlan: [I :WDI] WDTS_TxPacketComplete: Management frame Tx complete status: 0
18:49:39.867015  [18:49:39.858934] [00000016B70D4604] [VosTX] wlan: [I :WDA] Enter:WDA_TxComplete
18:49:39.867037  [18:49:39.858955] [00000016B70D4795] [VosTX] wlan: [I :VOS] VPKT [1428]: [0000000000000000] Packet returned, type 0[TX_802_11_MGMT]
18:49:39.867059  [18:49:39.859423] [00000016B70D6AB7] [VosMC] wlan: [I :SYS] tx_timer_deactivate() called for timer MIN CHANNEL TIMEOUT
18:49:39.867098  [18:49:39.859437] [00000016B70D6BBD] [VosMC] wlan: [IH:VOS] vos_timer_stop: Timer Addr inside voss_stop : 0x0000000000000000
18:49:39.867120  [18:49:39.859450] [00000016B70D6CBE] [VosMC] wlan: [IH:VOS] vos_timer_stop: Cannot stop timer in state = 19
18:49:39.867141  [18:49:39.859466] [00000016B70D6DEF] [VosMC] wlan: [I :SYS] Timer MIN CHANNEL TIMEOUT being activated
18:49:39.867180  [18:49:39.859478] [00000016B70D6EDD] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
18:49:39.867201  [18:49:39.859497] [00000016B70D7048] [VosMC] wlan: [I :SYS] Timer MIN CHANNEL TIMEOUT now activated
18:49:39.867240  [18:49:39.859509] [00000016B70D7124] [VosMC] wlan: [I :SYS] tx_timer_deactivate() called for timer MAX CHANNEL TIMEOUT
18:49:39.867279  [18:49:39.859520] [00000016B70D7202] [VosMC] wlan: [IH:VOS] vos_timer_stop: Timer Addr inside voss_stop : 0x0000000000000000
18:49:39.867301  [18:49:39.859532] [00000016B70D72E9] [VosMC] wlan: [IH:VOS] vos_timer_stop: Cannot stop timer in state = 19
18:49:39.867323  [18:49:39.859543] [00000016B70D73BF] [VosMC] wlan: [I :SYS] Timer MAX CHANNEL TIMEOUT being activated
18:49:39.867362  [18:49:39.859554] [00000016B70D748A] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
18:49:39.867405  [18:49:39.859567] [00000016B70D757E] [VosMC] wlan: [I :SYS] Timer MAX CHANNEL TIMEOUT now activated
18:49:39.867446  [18:49:39.859578] [00000016B70D765B] [VosMC] wlan: [I :SYS] tx_timer_deactivate() called for timer Periodic Probe Request Timer
18:49:39.867485  [18:49:39.859589] [00000016B70D7731] [VosMC] wlan: [IH:VOS] vos_timer_stop: Timer Addr inside voss_stop : 0x0000000000000000
18:49:39.867506  [18:49:39.859601] [00000016B70D7816] [VosMC] wlan: [IH:VOS] vos_timer_stop: Cannot stop timer in state = 19
18:49:39.867528  [18:49:39.859614] [00000016B70D7903] [VosMC] wlan: [I :SYS] Timer Periodic Probe Request Timer being activated		// 扫描请求定时器
18:49:39.867566  [18:49:39.859624] [00000016B70D79C9] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
18:49:39.867587  [18:49:39.859638] [00000016B70D7AD6] [VosMC] wlan: [I :SYS] Timer Periodic Probe Request Timer now activated
18:49:39.867626  [18:49:39.861202] [00000016B70DF068] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
18:49:39.867647  [18:49:39.861722] [00000016B70E1739] [swapp] wlan: [I :VOS] TIMER callback: running on MC thread
18:49:39.867668  [18:49:39.861831] [00000016B70E1F55] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS SYS MC Message queue
18:49:39.867689  [18:49:39.861850] [00000016B70E20B7] [VosMC] wlan: [I :SYS] Timer Periodic Probe Request Timer triggered			// 超时处理
18:49:39.867711  [18:49:39.861874] [00000016B70E2291] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS PE MC Message queue
18:49:39.867733  [18:49:39.861917] [00000016B70E25C3] [VosMC] wlan: [I :PE ] limProcessPeriodicProbeReqTimer: 4237: Mac Addr used in Probe Req is :da:a1:19:53:b9:f8
18:49:39.867755  [18:49:39.861994] [00000016B70E2B8E] [VosMC] wlan: [I :VOS] VPKT [779]: [0000000000000000] Packet allocated, type 0[TX_802_11_MGMT]
18:49:39.867776  [18:49:39.862042] [00000016B70E2F1F] [VosMC] wlan: [IH:WDA] Tx Mgmt Frame Subtype: 4 alloc(0000000000000000) txBdToken = 0
18:49:39.867799  [18:49:39.862070] [00000016B70E3141] [VosMC] wlan: [I :TL ] TL: using self sta addr to get staidx for spoofed probe req 00:0a:f5:32:b1:50
18:49:39.867820  [18:49:39.862086] [00000016B70E3278] [VosMC] wlan: [IL:TL ] WLAN TL: fProtMgmtFrame:0
18:49:39.867842  [18:49:39.862105] [00000016B70E33DB] [VosMC] wlan: [IL:TL ] WLAN TL: Dump TX meta info: txFlags:2, qosEnabled:0, ac:0, isEapol:0, fdisableFrmXlt:1, frmType:0
18:49:39.867863  [18:49:39.862118] [00000016B70E34DB] [VosMC] wlan: [IH:TL ] Serializing WDA TX Start Xmit event
18:49:39.867884  [18:49:39.862200] [00000016B70E3AFD] [VosTX] wlan: [I :VOS] VosTXThread: Servicing the VOS TL TX Message queue
18:49:39.875713  [18:49:39.862463] [00000016B70E4EBF] [VosTX] wlan: [I :WDI] WDTS_TxPacketComplete: Management frame Tx complete status: 0
18:49:39.875783  [18:49:39.862521] [00000016B70E5313] [VosTX] wlan: [I :WDA] Enter:WDA_TxComplete
18:49:39.875807  [18:49:39.862545] [00000016B70E54D5] [VosTX] wlan: [I :VOS] VPKT [1428]: [0000000000000000] Packet returned, type 0[TX_802_11_MGMT]
18:49:39.875829  [18:49:39.863277] [00000016B70E8BD9] [VosMC] wlan: [I :SYS] Timer Periodic Probe Request Timer being activated
18:49:39.875868  [18:49:39.863295] [00000016B70E8D1A] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
18:49:39.875890  [18:49:39.863315] [00000016B70E8E94] [VosMC] wlan: [I :SYS] Timer Periodic Probe Request Timer now activated
18:49:39.875929  [18:49:39.863568] [00000016B70EA19D] [VosRX] wlan: [I :VOS] VPKT [779]: [0000000000000000] Packet allocated, type 3[RX_RAW]
18:49:39.875951  [18:49:39.864074] [00000016B70EC78E] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS PE MC Message queue
18:49:39.875974  [18:49:39.864119] [00000016B70ECAE9] [VosMC] wlan: [I :PE ] limHandle80211Frames: 821: RX MGMT - Type 0, SubType 5,Seq.no 1927, Source mac-addr a0:04:60:8e:cf:8b
```

#### 被动扫描

```
13:51:17.028054  [13:51:16.916323] [0000000114D926EF] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS PE MC Message queue
13:51:17.028076  [13:51:16.916373] [0000000114D92AC0] [VosMC] wlan: [I :PE ] limContinueChannelScan: 1390: Current Channel to be scanned is 64
13:51:17.028099  [13:51:16.916372] [0000000114D92AB3] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
13:51:17.028122  [13:51:16.916407] [0000000114D92D3D] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS WDA MC Message queue
13:51:17.028144  [13:51:16.916423] [0000000114D92E71] [VosMC] wlan: [IL:WDA] =========> WDA_McProcessMsg msgType: 1037 
13:51:17.029844  [13:51:16.916450] [0000000114D93077] [VosMC] wlan: [I :WDA] ------> WDA_ProcessStartScanReq 
13:51:17.029880  [13:51:16.916557] [0000000114D9387D] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
13:51:17.029903  [13:51:16.923382] [0000000114DB38AE] [swapp] wlan: [I :VOS] TIMER callback: running on MC thread
13:51:17.029926  [13:51:16.923894] [0000000114DB5ECB] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS SYS MC Message queue
13:51:17.029971  [13:51:16.923929] [0000000114DB6168] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
13:51:17.029996  [13:51:16.927976] [0000000114DC9127] [VosMC] wlan: [IH:VOS] vos_timer_stop: Timer Addr inside voss_stop : 0x0000000000000000
13:51:17.030018  [13:51:16.928019] [0000000114DC9431] [VosMC] wlan: [I :WDA] <------ WDA_StartScanReqCallback 
13:51:17.030040  [13:51:16.928112] [0000000114DC9B29] [VosMC] wlan: [IH:VOS] vos_list_remove_front: list empty
13:51:17.030062  [13:51:16.928147] [0000000114DC9DC0] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS PE MC Message queue
13:51:17.030085  [13:51:16.928207] [0000000114DCA240] [VosMC] wlan: [I :PE ] limContinuePostChannelScan: 575: START PASSIVE Scan chan 64
13:51:17.030107  [13:51:16.928220] [0000000114DCA339] [VosMC] wlan: [I :SYS] tx_timer_deactivate() called for timer MAX CHANNEL TIMEOUT
13:51:17.030148  [13:51:16.928232] [0000000114DCA41D] [VosMC] wlan: [IH:VOS] vos_timer_stop: Timer Addr inside voss_stop : 0x0000000000000000
13:51:17.030170  [13:51:16.928244] [0000000114DCA50A] [VosMC] wlan: [IH:VOS] vos_timer_stop: Cannot stop timer in state = 19
13:51:17.030193  [13:51:16.928261] [0000000114DCA65A] [VosMC] wlan: [I :SYS] Timer MAX CHANNEL TIMEOUT being activated
13:51:17.030233  [13:51:16.928273] [0000000114DCA73C] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
13:51:17.030256  [13:51:16.928295] [0000000114DCA8DF] [VosMC] wlan: [I :SYS] Timer MAX CHANNEL TIMEOUT now activated
13:51:17.030296  [13:51:16.973733] [0000000114E9F8F8] [swapp] wlan: [I :VOS] TIMER callback: running on MC thread
13:51:17.030318  [13:51:16.974113] [0000000114EA154D] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS SYS MC Message queue
13:51:17.030341  [13:51:16.974148] [0000000114EA17D1] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
13:51:17.061520  [13:51:17.024632] [0000000114F8E270] [swapp] wlan: [I :VOS] TIMER callback: running on MC thread
13:51:17.061598  [13:51:17.024869] [0000000114F8F3E7] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS SYS MC Message queue
13:51:17.061623  [13:51:17.024903] [0000000114F8F67C] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
13:51:17.061647  [13:51:17.026933] [0000000114F98F03] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
13:51:17.061671  [13:51:17.033122] [0000000114FB5F31] [swapp] wlan: [I :VOS] TIMER callback: running on MC thread
13:51:17.061694  [13:51:17.033292] [0000000114FB6BA3] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS SYS MC Message queue
13:51:17.061716  [13:51:17.033311] [0000000114FB6D0E] [VosMC] wlan: [I :SYS] Timer MAX CHANNEL TIMEOUT triggered
13:51:17.061739  [13:51:17.033337] [0000000114FB6F05] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS PE MC Message queue
13:51:17.061762  [13:51:17.033368] [0000000114FB714E] [VosMC] wlan: [I :PE ] limProcessMaxChannelTimeout: 4127: Scanning : Max channel timed out
13:51:17.061784  [13:51:17.033382] [0000000114FB7262] [VosMC] wlan: [I :SYS] tx_timer_deactivate() called for timer MAX CHANNEL TIMEOUT
13:51:17.061825  [13:51:17.033399] [0000000114FB73AE] [VosMC] wlan: [IH:VOS] vos_timer_stop: Timer Addr inside voss_stop : 0x0000000000000000
13:51:17.061846  [13:51:17.033412] [0000000114FB749F] [VosMC] wlan: [IH:VOS] vos_timer_stop: Cannot stop timer in state = 19
13:51:17.061869  [13:51:17.033427] [0000000114FB75C9] [VosMC] wlan: [I :SYS] tx_timer_deactivate() called for timer Periodic Probe Request Timer
13:51:17.061909  [13:51:17.033438] [0000000114FB769F] [VosMC] wlan: [IH:VOS] vos_timer_stop: Timer Addr inside voss_stop : 0x0000000000000000
13:51:17.061931  [13:51:17.033450] [0000000114FB7783] [VosMC] wlan: [IH:VOS] vos_timer_stop: Cannot stop timer in state = 19
13:51:17.061955  [13:51:17.033468] [0000000114FB78D4] [VosMC] wlan: [W :PE ] limProcessMaxChannelTimeout: 4154: Sending End Scan req from MAX_CH_TIMEOUT in state 2 on ch-64
```

### kernel代码分析

```
驱动模块初始化函数
vendor\qcom\opensource\wlan\prima\CORE\HDD\src\wlan_hdd_main.c
	static int hdd_driver_init( void)

vendor\qcom\opensource\wlan\qcacld-2.0\CORE\HDD\src\wlan_hdd_main.c
int hdd_wlan_startup(struct device *dev, v_VOID_t *hif_sc)
	status = vos_open( &pVosContext, 0);

vendor\qcom\opensource\wlan\prima\CORE\VOSS\src\vos_api.c
	VOS_STATUS vos_open( v_CONTEXT_t *pVosContext, void *devHandle )

其中开启线程
vendor\qcom\opensource\wlan\prima\CORE\VOSS\src\vos_sched.c
VOS_STATUS
vos_sched_open()
{
	// 创建线程
	//Create the VOSS Main Controller thread
	pSchedContext->McThread = kthread_create(VosMCThread, pSchedContext,
                                           "VosMCThread");
}
										   
// 线程函数
vendor\qcom\opensource\wlan\prima\CORE\MAC\src\pe\lim\limProcessMlmReqMessages.c
static int
VosMCThread
(
  void * Arg
)
{

  set_user_nice(current, -2);

  while(!shutdown)
  {
    // This implements the execution model algorithm
    retWaitStatus = wait_event_interruptible(pSchedContext->mcWaitQueue,
       test_bit(MC_POST_EVENT, &pSchedContext->mcEventFlag) ||
       test_bit(MC_SUSPEND_EVENT, &pSchedContext->mcEventFlag));

    if(retWaitStatus == -ERESTARTSYS)
    {
      VOS_TRACE(VOS_MODULE_ID_VOSS, VOS_TRACE_LEVEL_ERROR,
         "%s: wait_event_interruptible returned -ERESTARTSYS", __func__);
      break;
    }
    clear_bit(MC_POST_EVENT, &pSchedContext->mcEventFlag);
	// 循环
    while(1)
    {
      // Check if MC needs to shutdown
      if(test_bit(MC_SHUTDOWN_EVENT, &pSchedContext->mcEventFlag))
      {
        VOS_TRACE(VOS_MODULE_ID_VOSS, VOS_TRACE_LEVEL_INFO,
                "%s: MC thread signaled to shutdown", __func__);
        shutdown = VOS_TRUE;
        /* Check for any Suspend Indication */
        if (test_and_clear_bit(MC_SUSPEND_EVENT,
                               &pSchedContext->mcEventFlag))
        {
           /* Unblock anyone waiting on suspend */
           complete(&pHddCtx->mc_sus_event_var);
        }
        break;
      }
 
		// while循环中处理各种消息
        vStatus = WDA_McProcessMsg( pSchedContext->pVContext, pMsgWrapper->pVosMsg);
		

vendor\qcom\opensource\wlan\prima\CORE\WDA\src\wlan_qct_wda.c
VOS_STATUS WDA_McProcessMsg( v_CONTEXT_t pVosContext, vos_msg_t *pMsg )
{
   /* Process all the WDA messages.. */
   switch( pMsg->type )
   {
		// 扫描初始化
      case WDA_INIT_SCAN_REQ:
      {
         WDA_ProcessInitScanReq(pWDA, (tInitScanParams *)pMsg->bodyptr) ;
         break ;    
      }
      /* start SCAN request from PE */
      case WDA_START_SCAN_REQ:				// 扫描
      {
         WDA_ProcessStartScanReq(pWDA, (tStartScanParams *)pMsg->bodyptr) ;
         break ;    
      }
	  
	  
// 开始扫描请求
vendor\qcom\opensource\wlan\prima\CORE\WDA\src\wlan_qct_wda.c
VOS_STATUS  WDA_ProcessStartScanReq(tWDA_CbContext *pWDA, 
                                           tStartScanParams *startScanParams)
{

   // 开始扫描, 使用回调函数,调用WDA_StartScanReqCallback.
   status = WDI_StartScanReq(wdiStartScanParams, 
                              WDA_StartScanReqCallback, pWdaParams) ;
   /* failure returned by WDI API */
   if(IS_WDI_STATUS_FAILURE(status))
   {
      VOS_TRACE( VOS_MODULE_ID_WDA, VOS_TRACE_LEVEL_ERROR,
                     "Failure in Start Scan WDI API, free all the memory "
                     "It should be due to previous abort scan." );
      vos_mem_free(pWdaParams->wdaWdiApiMsgParam);
      vos_mem_free(pWdaParams) ;
      startScanParams->status = eSIR_FAILURE ;
      WDA_SendMsg(pWDA, WDA_START_SCAN_RSP, (void *)startScanParams, 0) ;		
   }
   return CONVERT_WDI2VOS_STATUS(status) ;
}

vendor\qcom\opensource\wlan\prima\CORE\WDA\src\wlan_qct_wda.c
void WDA_StartScanReqCallback(WDI_StartScanRspParamsType *pScanRsp, 
                                                    void* pUserData)
{
   // 发送WDA_START_SCAN_RSP消息
   /* assign status to scan params */
   pWDA_ScanParam->status = pScanRsp->wdiStatus ;
   /* send SCAN RSP message back to PE */
   WDA_SendMsg(pWDA, WDA_START_SCAN_RSP, (void *)pWDA_ScanParam, 0) ;				
   return ;
}

vendor\qcom\opensource\wlan\prima\CORE\MAC\src\pe\lim\limProcessMessageQueue.c
void
limProcessMessages(tpAniSirGlobal pMac, tpSirMsgQ  limMsg)
{
     case SIR_LIM_UPDATE_BEACON:
            limUpdateBeacon(pMac);		// 更新beacon
			break;
     case WDA_INIT_SCAN_RSP:
            limProcessInitScanRsp(pMac, limMsg->bodyptr);
            limMsg->bodyptr = NULL;
            break;
	 case WDA_START_SCAN_RSP:				// 处理消息
            limProcessStartScanRsp(pMac, limMsg->bodyptr);
            limMsg->bodyptr = NULL;
            break;

			
// beacon处理函数。
vendor\qcom\opensource\wlan\prima\CORE\MAC\src\pe\lim\limUtils.c

			
vendor\qcom\opensource\wlan\prima\CORE\MAC\src\pe\lim\limProcessMlmRspMessages.c
void limProcessStartScanRsp(tpAniSirGlobal pMac,  void *body)
{
    tpStartScanParams       pStartScanParam;
    eHalStatus              status;
    SET_LIM_PROCESS_DEFD_MESGS(pMac, true);
    pStartScanParam = (tpStartScanParams) body;
    status = pStartScanParam->status;
#if defined WLAN_FEATURE_VOWIFI
    //HAL fills in the tx power used for mgmt frames in this field.
    //Store this value to use in TPC report IE.
    rrmCacheMgmtTxPower( pMac, pStartScanParam->txMgmtPower, NULL );
    //Store start TSF of scan start. This will be stored in BSS params.
    rrmUpdateStartTSF( pMac, pStartScanParam->startTSF );
#endif
    vos_mem_free(body);
    body = NULL;
    if( pMac->lim.abortScan )
    {
        limLog( pMac, LOGW, FL(" finish scan") );
        pMac->lim.abortScan = 0;
        limDeactivateAndChangeTimer(pMac, eLIM_MIN_CHANNEL_TIMER);
        limDeactivateAndChangeTimer(pMac, eLIM_MAX_CHANNEL_TIMER);
        //Set the resume channel to Any valid channel (invalid). 
        //This will instruct HAL to set it to any previous valid channel.
        peSetResumeChannel(pMac, 0, 0);
        limSendHalFinishScanReq(pMac, eLIM_HAL_FINISH_SCAN_WAIT_STATE);
    }
    switch(pMac->lim.gLimHalScanState)
    {
        case eLIM_HAL_START_SCAN_WAIT_STATE:
            if (status != (tANI_U32) eHAL_STATUS_SUCCESS)
            {
               PELOGW(limLog(pMac, LOGW, FL("StartScanRsp with failed status= %d"), status);)
               //
               // FIXME - With this, LIM will try and recover state, but
               // eWNI_SME_SCAN_CNF maybe reporting an incorrect
               // status back to the SME
               //
               //Set the resume channel to Any valid channel (invalid). 
               //This will instruct HAL to set it to any previous valid channel.
               peSetResumeChannel(pMac, 0, 0);
               limSendHalFinishScanReq( pMac, eLIM_HAL_FINISH_SCAN_WAIT_STATE );
               //limCompleteMlmScan(pMac, eSIR_SME_HAL_SCAN_INIT_FAILED);
            }
            else
            {
               pMac->lim.gLimHalScanState = eLIM_HAL_SCANNING_STATE;
               limContinuePostChannelScan(pMac);						// 扫描
            }
            break;
        default:
            limLog(pMac, LOGW, FL("Rcvd StartScanRsp not in WAIT State, state %d"),
                     pMac->lim.gLimHalScanState);
            break;
    }
    return;
}

vendor\qcom\opensource\wlan\prima\CORE\MAC\src\pe\lim\limProcessMlmReqMessages.c
void limContinuePostChannelScan(tpAniSirGlobal pMac)
{
// 开始扫描
       do
        {
            tSirMacAddr         gSelfMacAddr;

            /* Send self MAC as src address if
             * MAC spoof is not enabled OR
             * spoofMacAddr is all 0 OR
             * disableP2PMacSpoof is enabled and scan is P2P scan
             * else use the spoofMac as src address
             */
            if ((pMac->lim.isSpoofingEnabled != TRUE) ||
                (TRUE ==
                vos_is_macaddr_zero((v_MACADDR_t *)&pMac->lim.spoofMacAddr)) ||
                (pMac->roam.configParam.disableP2PMacSpoofing &&
                pMac->lim.gpLimMlmScanReq->p2pSearch)) {
                vos_mem_copy(gSelfMacAddr, pMac->lim.gSelfMacAddr, VOS_MAC_ADDRESS_LEN);
            } else {
                vos_mem_copy(gSelfMacAddr, pMac->lim.spoofMacAddr, VOS_MAC_ADDRESS_LEN);
            }
            limLog(pMac, LOG1,
                 FL(" Mac Addr "MAC_ADDRESS_STR " used in sending ProbeReq number %d, for SSID %s on channel: %d"),
                      MAC_ADDR_ARRAY(gSelfMacAddr) ,i, pMac->lim.gpLimMlmScanReq->ssId[i].ssId, channelNum);
            // include additional IE if there is
// 发出probe request数据帧，主动扫描
            status = limSendProbeReqMgmtFrame( pMac, &pMac->lim.gpLimMlmScanReq->ssId[i],
               pMac->lim.gpLimMlmScanReq->bssId, channelNum, gSelfMacAddr,
               pMac->lim.gpLimMlmScanReq->dot11mode,
               pMac->lim.gpLimMlmScanReq->uIEFieldLen,
               (tANI_U8 *)(pMac->lim.gpLimMlmScanReq)+pMac->lim.gpLimMlmScanReq->uIEFieldOffset);
            
            if ( status != eSIR_SUCCESS)
            {
                PELOGE(limLog(pMac, LOGE, FL("send ProbeReq failed for SSID %s on channel: %d"),
                                                pMac->lim.gpLimMlmScanReq->ssId[i].ssId, channelNum);)
                limDeactivateAndChangeTimer(pMac, eLIM_MIN_CHANNEL_TIMER);
                limSendHalEndScanReq(pMac, channelNum, eLIM_HAL_END_SCAN_WAIT_STATE);
                return;
            }
            i++;
        } while (i < pMac->lim.gpLimMlmScanReq->numSsid);
   else
    {
        tANI_U32 val;
// 被动扫描
        limLog(pMac, LOG1, FL("START PASSIVE Scan chan %d"), channelNum);				

        /// Passive Scanning. Activate maxChannelTimer
        if (tx_timer_deactivate(&pMac->lim.limTimers.gLimMaxChannelTimer)				// 定时器, 信道停留最大时间
                                      != TX_SUCCESS)
        {
            // Could not deactivate max channel timer.
            // Log error
            limLog(pMac, LOGE, FL("Unable to deactivate max channel timer"));
            limSendHalEndScanReq(pMac, channelNum,
                                 eLIM_HAL_END_SCAN_WAIT_STATE);
        }
        else
        {
            if (pMac->miracast_mode)
            {
                val = DEFAULT_MIN_CHAN_TIME_DURING_MIRACAST +
                    DEFAULT_MAX_CHAN_TIME_DURING_MIRACAST;
            }
            else if (wlan_cfgGetInt(pMac, WNI_CFG_PASSIVE_MAXIMUM_CHANNEL_TIME,
                          &val) != eSIR_SUCCESS)
            {
                /**
                 * Could not get max channel value
                 * from CFG. Log error.
                 */
                limLog(pMac, LOGE,
                 FL("could not retrieve passive max chan value, Use Def val"));
                val= WNI_CFG_PASSIVE_MAXIMUM_CHANNEL_TIME_STADEF;
            }

            val = SYS_MS_TO_TICKS(val);
			// 重新设置定时器
            if (tx_timer_change(&pMac->lim.limTimers.gLimMaxChannelTimer,		
                        val, 0) != TX_SUCCESS)
            {
                // Could not change max channel timer.
                // Log error
                limLog(pMac, LOGE, FL("Unable to change max channel timer"));
                limDeactivateAndChangeTimer(pMac, eLIM_MAX_CHANNEL_TIMER);
                limSendHalEndScanReq(pMac, channelNum,
                                  eLIM_HAL_END_SCAN_WAIT_STATE);
                return;
            }
            else if (tx_timer_activate(&pMac->lim.limTimers.gLimMaxChannelTimer)
                                                                  != TX_SUCCESS)
            {

                limLog(pMac, LOGE, FL("could not start max channel timer"));
                limDeactivateAndChangeTimer(pMac, eLIM_MAX_CHANNEL_TIMER);
                limSendHalEndScanReq(pMac, channelNum,
                                 eLIM_HAL_END_SCAN_WAIT_STATE);
                return;
            }
        }
        // Wait for Beacons to arrive
    } // if (pMac->lim.gLimMlmScanReq->scanType == eSIR_ACTIVE_SCAN)

    limAddScanChannelInfo(pMac, channelNum);
    return;
}

probe request位置
vendor\qcom\opensource\wlan\prima\CORE\MAC\src\pe\lim\limSendManagementFrames.c

停止定时器处理函数
vendor\qcom\opensource\wlan\prima\CORE\SYS\legacy\src\platform\src\VossWrapper.c

超时处理函数, 代码中可以看到，好多超时用的都是同一个函数
vendor\qcom\opensource\wlan\prima\CORE\MAC\src\pe\lim\limProcessMessageQueue.c
void
limProcessMessages(tpAniSirGlobal pMac, tpSirMsgQ  limMsg)
        case SIR_LIM_MIN_CHANNEL_TIMEOUT:
        case SIR_LIM_MAX_CHANNEL_TIMEOUT:
        case SIR_LIM_PERIODIC_PROBE_REQ_TIMEOUT:
        case SIR_LIM_JOIN_FAIL_TIMEOUT:
        case SIR_LIM_PERIODIC_JOIN_PROBE_REQ_TIMEOUT:
        case SIR_LIM_AUTH_FAIL_TIMEOUT:
        case SIR_LIM_AUTH_RSP_TIMEOUT:
        case SIR_LIM_ASSOC_FAIL_TIMEOUT:
        case SIR_LIM_REASSOC_FAIL_TIMEOUT:
#ifdef WLAN_FEATURE_VOWIFI_11R
        case SIR_LIM_FT_PREAUTH_RSP_TIMEOUT:
#endif
        case SIR_LIM_REMAIN_CHN_TIMEOUT:
        case SIR_LIM_INSERT_SINGLESHOT_NOA_TIMEOUT:
        case SIR_LIM_DISASSOC_ACK_TIMEOUT:
        case SIR_LIM_DEAUTH_ACK_TIMEOUT:
        case SIR_LIM_CONVERT_ACTIVE_CHANNEL_TO_PASSIVE:
        case SIR_LIM_AUTH_RETRY_TIMEOUT:
        case SIR_LIM_SAP_ECSA_TIMEOUT:
#ifdef WLAN_FEATURE_LFR_MBB
        case SIR_LIM_PREAUTH_MBB_RSP_TIMEOUT:
        case SIR_LIM_REASSOC_MBB_RSP_TIMEOUT:
#endif
            // These timeout messages are handled by MLM sub module

            limProcessMlmReqMessages(pMac,
                                     limMsg);

            break;
}

不同消息函数处理
vendor\qcom\opensource\wlan\prima\CORE\MAC\src\pe\lim\limProcessMlmReqMessages.c
void
limProcessMlmReqMessages(tpAniSirGlobal pMac, tpSirMsgQ Msg)
{
    switch (Msg->type)
    {
        case LIM_MLM_START_REQ:             limProcessMlmStartReq(pMac, Msg->bodyptr);   break;
        case LIM_MLM_SCAN_REQ:              limProcessMlmScanReq(pMac, Msg->bodyptr);    break;
#ifdef FEATURE_OEM_DATA_SUPPORT
        case LIM_MLM_OEM_DATA_REQ: limProcessMlmOemDataReq(pMac, Msg->bodyptr); break;
#endif
        case LIM_MLM_JOIN_REQ:              limProcessMlmJoinReq(pMac, Msg->bodyptr);    break;
        case LIM_MLM_AUTH_REQ:              limProcessMlmAuthReq(pMac, Msg->bodyptr);    break;
        case LIM_MLM_ASSOC_REQ:             limProcessMlmAssocReq(pMac, Msg->bodyptr);   break;
        // 重关联超时
		case LIM_MLM_REASSOC_REQ:           limProcessMlmReassocReq(pMac, Msg->bodyptr); break;
        // 取消关联请求
		case LIM_MLM_DISASSOC_REQ:          limProcessMlmDisassocReq(pMac, Msg->bodyptr);  break;
        case LIM_MLM_DEAUTH_REQ:            limProcessMlmDeauthReq(pMac, Msg->bodyptr);  break;
        case LIM_MLM_SETKEYS_REQ:           limProcessMlmSetKeysReq(pMac, Msg->bodyptr);  break;
        case LIM_MLM_REMOVEKEY_REQ:         limProcessMlmRemoveKeyReq(pMac, Msg->bodyptr); break;
        case SIR_LIM_MIN_CHANNEL_TIMEOUT:   limProcessMinChannelTimeout(pMac);  break;
        // 超时处理
		case SIR_LIM_MAX_CHANNEL_TIMEOUT:   limProcessMaxChannelTimeout(pMac);  break;
        case SIR_LIM_PERIODIC_PROBE_REQ_TIMEOUT:
                               limProcessPeriodicProbeReqTimer(pMac);  break;
        case SIR_LIM_JOIN_FAIL_TIMEOUT:     limProcessJoinFailureTimeout(pMac);  break;
        case SIR_LIM_PERIODIC_JOIN_PROBE_REQ_TIMEOUT:
                                            limProcessPeriodicJoinProbeReqTimer(pMac); break;
        case SIR_LIM_AUTH_FAIL_TIMEOUT:     limProcessAuthFailureTimeout(pMac);  break;
        case SIR_LIM_AUTH_RSP_TIMEOUT:      limProcessAuthRspTimeout(pMac, Msg->bodyval);  break;
        case SIR_LIM_ASSOC_FAIL_TIMEOUT:    limProcessAssocFailureTimeout(pMac, Msg->bodyval);  break;
#ifdef WLAN_FEATURE_VOWIFI_11R
        case SIR_LIM_FT_PREAUTH_RSP_TIMEOUT:limProcessFTPreauthRspTimeout(pMac); break;
#endif
#ifdef WLAN_FEATURE_LFR_MBB
        case SIR_LIM_PREAUTH_MBB_RSP_TIMEOUT:
             lim_process_preauth_mbb_rsp_timeout(pMac);
             break;
        case SIR_LIM_REASSOC_MBB_RSP_TIMEOUT:
             lim_process_reassoc_mbb_rsp_timeout(pMac);
             break;
#endif
        case SIR_LIM_REMAIN_CHN_TIMEOUT:    limProcessRemainOnChnTimeout(pMac); break;
        case SIR_LIM_INSERT_SINGLESHOT_NOA_TIMEOUT:   
                                            limProcessInsertSingleShotNOATimeout(pMac); break;
        case SIR_LIM_CONVERT_ACTIVE_CHANNEL_TO_PASSIVE:
                                            limConvertActiveChannelToPassiveChannel(pMac); break;
        case SIR_LIM_AUTH_RETRY_TIMEOUT:
                                            limProcessAuthRetryTimer(pMac);
                                            break;
        case SIR_LIM_DISASSOC_ACK_TIMEOUT:  limProcessDisassocAckTimeout(pMac); break;
        case SIR_LIM_DEAUTH_ACK_TIMEOUT:    limProcessDeauthAckTimeout(pMac); break;
        case SIR_LIM_SAP_ECSA_TIMEOUT:      lim_process_ap_ecsa_timeout(pMac);break;
        case LIM_MLM_ADDBA_REQ:             limProcessMlmAddBAReq( pMac, Msg->bodyptr ); break;
        case LIM_MLM_ADDBA_RSP:             limProcessMlmAddBARsp( pMac, Msg->bodyptr ); break;
        case LIM_MLM_DELBA_REQ:             limProcessMlmDelBAReq( pMac, Msg->bodyptr ); break;
        case LIM_MLM_TSPEC_REQ:                 
        default:
            break;
    } // switch (msgType)
} /*** end limProcessMlmReqMessages() ***/


vendor\qcom\opensource\wlan\prima\CORE\MAC\src\pe\lim\limProcessMlmReqMessages.c
void
limProcessMlmReqMessages(tpAniSirGlobal pMac, tpSirMsgQ Msg)

vendor\qcom\opensource\wlan\prima\CORE\MAC\src\pe\lim\limProcessMlmReqMessages.c
超时函数
static void
limProcessMaxChannelTimeout(tpAniSirGlobal pMac)
{
    tANI_U8 channelNum;

    /*do not process if we are in finish scan wait state i.e.
     scan is aborted or finished*/
    if ((pMac->lim.gLimMlmState == eLIM_MLM_WT_PROBE_RESP_STATE ||
        pMac->lim.gLimMlmState == eLIM_MLM_PASSIVE_SCAN_STATE) &&
        pMac->lim.gLimHalScanState != eLIM_HAL_FINISH_SCAN_WAIT_STATE)
    {
        limLog(pMac, LOG1, FL("Scanning : Max channel timed out"));			// 打印log，提示超时
        /**
         * MAX channel timer timed out
         * Continue channel scan.
         */
        limDeactivateAndChangeTimer(pMac, eLIM_MAX_CHANNEL_TIMER);
        limDeactivateAndChangeTimer(pMac, eLIM_PERIODIC_PROBE_REQ_TIMER);
        pMac->lim.limTimers.gLimPeriodicProbeReqTimer.sessionId = 0xff;
        pMac->lim.probeCounter = 0;

       if (pMac->lim.gLimCurrentScanChannelId <=
                (tANI_U32)(pMac->lim.gpLimMlmScanReq->channelList.numChannels - 1))
        {
        channelNum = limGetCurrentScanChannel(pMac);
        }
        else
        {
            if(pMac->lim.gpLimMlmScanReq->channelList.channelNumber)
            {
                channelNum = pMac->lim.gpLimMlmScanReq->channelList.channelNumber[pMac->lim.gpLimMlmScanReq->channelList.numChannels - 1];
            }
            else
            {
               channelNum = 1;
            }
        }
        limLog(pMac, LOGW,
           FL("Sending End Scan req from MAX_CH_TIMEOUT in state %d on ch-%d"),		// kerel log
           pMac->lim.gLimMlmState,channelNum);
        limSendHalEndScanReq(pMac, channelNum, eLIM_HAL_END_SCAN_WAIT_STATE);		// 停止扫描请求
    }
    else
    {
        /**
         * MAX channel timer should not have timed out
         * in states other than wait_scan.
         * Log error.
         */
        limLog(pMac, LOGW,
           FL("received unexpected MAX channel timeout in mlme state %d and hal scan state %d"),
           pMac->lim.gLimMlmState, pMac->lim.gLimHalScanState);
        limPrintMlmState(pMac, LOGW, pMac->lim.gLimMlmState);
    }
} /*** limProcessMaxChannelTimeout() ***/
```