记录一下高通对于beacon loss的处理的日志。方便后面再来分析。

```
04:17:13.397139  [04:17:13.383909] [000000A81699E7EC] [VosMC] wlan: [W :PMC] pmmExitBmpsResponseHandler: 882: Received SIR_HAL_EXIT_BMPS_RSP in correct state. 
04:17:13.397150  [04:17:13.384823] [000000A8169A2C98] [VosMC] wlan: [W :PMC] pmmExitBmpsResponseHandler: 927: Rcvd SIR_HAL_EXIT_BMPS_RSP with MISSED_BEACON
04:17:13.397160  [04:17:13.384841] [000000A8169A2DC5] [VosMC] wlan: [I :PMC] pmmMissedBeaconHandler: 972: The device woke up due to MISSED BEACON 
04:17:13.397171  [04:17:13.385539] [000000A8169A624C] [VosMC] wlan: [W :PE ] limHandleHeartBeatFailure: 518: Heartbeat Failure
04:17:13.397182  [04:17:13.385559] [000000A8169A63AB] [VosMC] wlan: [W :PE ] limHandleHeartBeatFailure: 536: Heart Beat missed from AP. Sending Probe Req
04:17:13.397194  [04:17:13.385667] [000000A8169A6BCE] [VosMC] wlan: [IH:WDA] Tx Mgmt Frame Subtype: 4 alloc(0000000000000000) txBdToken = 0
04:17:13.397204  [04:17:13.386531] [000000A8169AACA9] [VosTX] wlan: [I :WDI] WDTS_TxPacketComplete: Management frame Tx complete status: 0
04:17:13.397215  [04:17:13.386555] [000000A8169AAE58] [VosTX] wlan: [I :WDA] Enter:WDA_TxComplete
04:17:13.397225  [04:17:13.387076] [000000A8169AD59C] [VosMC] wlan: [W :PE ] limHandleHeartBeatTimeout: 7376: Sending Probe for Session: 0
04:17:13.397248  [04:17:13.387082] [000000A8169AD5E1] [VosRX] wlan: [I :HDD] STA RX ARP
04:17:13.397260  [04:17:13.387096] [000000A8169AD6E9] [VosRX] wlan: [I :HDD] hdd_rx_packet_cbk :STA RX ARP received
04:17:13.397271  [04:17:13.387104] [000000A8169AD790] [VosMC] wlan: [I :PE ] limDeactivateAndChangeTimer: 1545: Deactivated probe after hb timer
04:17:13.397281  [04:17:13.387124] [000000A8169AD909] [VosMC] wlan: [W :PE ] limDeactivateAndChangeTimer: 1572: Probe after HB timer value is changed = 7
04:17:13.397292  [04:17:13.387210] [000000A8169ADF71] [VosMC] wlan: [IH:PMC] pmcMessageProcessor: 1632: Message type 5763
04:17:13.397302  [04:17:13.387229] [000000A8169AE0E3] [VosMC] wlan: [IH:PMC] pmcProcessResponse: 1250: process message = 0x1683
04:17:13.397313  [04:17:13.387245] [000000A8169AE210] [VosMC] wlan: [IH:PMC] pmcProcessResponse: 1421: Rcvd eWNI_PMC_EXIT_BMPS_RSP with status = 0
04:17:13.397324  [04:17:13.387262] [000000A8169AE361] [VosMC] wlan: [I :PMC] pmcEnterFullPowerState: 166: PMC state is 7
04:17:13.397334  [04:17:13.387262] [000000A8169AE372] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.397345  [04:17:13.387284] [000000A8169AE50E] [VosMC] wlan: [I :PMC] pmcStartTrafficTimer: 886: Enter. Timer duration is 1000
04:17:13.496823  [04:17:13.387307] [000000A8169AE6BB] [VosMC] wlan: [IH:PMC] pmcDoCallbacks: 844: Enter
04:17:13.496896  [04:17:13.387375] [000000A8169AEBE4] [VosMC] wlan: [I :PMC] pmcDoDeviceStateUpdateCallbacks: 1556: PMC - Update registered modules of new device state: FULL_POWER
04:17:13.496908  [04:17:13.387394] [000000A8169AED45] [VosMC] wlan: [IH:SME] sme_QosProcessOutOfUapsdMode: 7473: Flow List empty, can't search
04:17:13.496919  [04:17:13.387409] [000000A8169AEE69] [VosMC] wlan: [IH:SME] sme_QosPmcDeviceStateUpdateInd: 7447: ignoring Device(PMC) state change to FULL_POWER (1)
04:17:13.496931  [04:17:13.387428] [000000A8169AEFD9] [VosRX] wlan: [I :HDD] __hdd_hard_start_xmit :ARP packet received form net_dev
04:17:13.496942  [04:17:13.387435] [000000A8169AF05D] [VosMC] wlan: [I :HDD] Not in IMPS/BMPS and suspended state
04:17:13.496953  [04:17:13.387471] [000000A8169AF317] [VosMC] wlan: [I :HDD] hdd_conf_mcastbcast_filter: Configuring Mcast/Bcast Filter Setting. setfilter 0
04:17:13.496965  [04:17:13.387495] [000000A8169AF4DB] [VosMC] wlan: [I :HDD] Success to post set/reset filter tolower mac with status 0configuredMcstBcstFilterSetting = 3setMcstBcstFilter = 0
04:17:13.496975  [04:17:13.387508] [000000A8169AF5CA] [VosMC] wlan: [I :PMC] PMC: Enter full power done
04:17:13.496985  [04:17:13.387569] [000000A8169AFA72] [VosRX] wlan: [I :HDD] STA RX ARP packet Delivered to net stack
04:17:13.496996  [04:17:13.387579] [000000A8169AFB25] [VosMC] wlan: [IL:WDA] =========> WDA_McProcessMsg msgType: 10bb 
04:17:13.497007  [04:17:13.387614] [000000A8169AFDCE] [VosMC] wlan: [I :WDA] ------> WDA_ProcessConfigureRxpFilterReq 
04:17:13.497018  [04:17:13.387674] [000000A8169B024E] [VosRX] wlan: [I :HDD] STA RX ARP
04:17:13.497028  [04:17:13.387689] [000000A8169B0360] [VosRX] wlan: [I :HDD] hdd_rx_packet_cbk :STA RX ARP received
04:17:13.497039  [04:17:13.387751] [000000A8169B080E] [VosMC] wlan: [I :WDA] <------ WDA_ConfigureRxpFilterReqCallback, wdiStatus: 0
04:17:13.497051  [04:17:13.387790] [000000A8169B0AF7] [VosRX] wlan: [I :HDD] __hdd_hard_start_xmit :ARP packet received form net_dev
04:17:13.497062  [04:17:13.387843] [000000A8169B0EF1] [VosRX] wlan: [I :HDD] STA RX ARP packet Delivered to net stack
04:17:13.497073  [04:17:13.388067] [000000A8169B1FFB] [VosTX] wlan: [I :HDD] STA TX ARP
04:17:13.497084  [04:17:13.388089] [000000A8169B215E] [VosTX] wlan: [I :HDD] hdd_tx_fetch_packet_cbk :STA TX ARP Received in TL 
04:17:13.497094  [04:17:13.388170] [000000A8169B277C] [VosTX] wlan: [I :HDD] STA TX ARP
04:17:13.497105  [04:17:13.388182] [000000A8169B2860] [VosTX] wlan: [I :HDD] hdd_tx_fetch_packet_cbk :STA TX ARP Received in TL 
04:17:13.497118  [04:17:13.388279] [000000A8169B2FB2] [VosTX] wlan: [I :HDD] WDTS_TxPacket :Transmitting ARP packet
04:17:13.497129  [04:17:13.389007] [000000A8169B6649] [VosMC] wlan: [I :WDA] <------ WDA_ConfigureRxpFilterRespCallback 
04:17:13.497177  [04:17:13.389012] [000000A8169B66A2] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.497190  [04:17:13.392998] [000000A8169C91A3] [VosTX] wlan: [I :HDD] WDTS_TxPacket :Transmitting ARP packet
04:17:13.497201  [04:17:13.394036] [000000A8169CDF84] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.497214  [04:17:13.395245] [000000A8169D3A2D] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.497227  [04:17:13.396589] [000000A8169D9EED] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.497237  [04:17:13.466327] [000000A816B20D64] [VosMC] wlan: [E :PE ] limHandleHeartBeatFailureTimeout: 7522: Probe_hb_failure: SME 12, MLME 16, HB-Count 0 BCN count 2
04:17:13.497249  [04:17:13.481214] [000000A816B669C4] [VosMC] wlan: [E :PE ] limHandleHeartBeatFailureTimeout: 7539: Probe_hb_failure: for session:0 
04:17:13.497261  [04:17:13.482682] [000000A816B6D81A] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.497272  [04:17:13.483725] [000000A816B72629] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.497283  [04:17:13.494033] [000000A816BA2B69] [VosMC] wlan: [W :PE ] limTearDownLinkWithAp: 386: No ProbeRsp from AP after HB failure. Tearing down link
04:17:13.503788  [04:17:13.494104] [000000A816BA308C] [VosMC] wlan: [I :PE ] limProcessMlmDeauthInd: 1533: *** Received Deauthentication from staId=23130 ***
04:17:13.503828  [04:17:13.494724] [000000A816BA5F16] [VosMC] wlan: [IL:WDA] =========> WDA_McProcessMsg msgType: 113d 
04:17:13.503839  [04:17:13.494743] [000000A816BA6073] [VosMC] wlan: [I :WDA] WDA_ProcessTLPauseInd: 16010: ---> WDA_ProcessTLPauseInd
04:17:13.503850  [04:17:13.494844] [000000A816BA681A] [VosMC] wlan: [IH:HDD] CSR Callback: status= 31 result= 46 roamID=0
04:17:13.503860  [04:17:13.494867] [000000A816BA69B8] [VosMC] wlan: [IH:HDD] hdd_tdlsStatusUpdate: DEL_ALL_TDLS_PEER_IND staIdx 0 00:00:00:00:00:00
04:17:13.503871  [04:17:13.494904] [000000A816BA6C87] [VosMC] wlan: [W :HDD] wlan_hdd_tdls_check_bmps: No TDLS peer connected/discovery sent. Enable BMPS
04:17:13.503882  [04:17:13.495228] [000000A816BA84F4] [VosMC] wlan: [IH:PMC] pmcEnablePowerSave: 636: Entering pmcEnablePowerSave, power save mode 1
04:17:13.503892  [04:17:13.495707] [000000A816BAA8EC] [VosMC] wlan: [IH:PMC] pmcStartAutoBmpsTimer: 709: Entering pmcStartAutoBmpsTimer
04:17:13.503902  [04:17:13.495733] [000000A816BAAAAE] [VosMC] wlan: [I :PMC] pmcStartTrafficTimer: 886: Enter. Timer duration is 1000
04:17:13.503913  [04:17:13.495749] [000000A816BAABEC] [VosMC] wlan: [W :PMC] pmcStartTrafficTimer: 894:   traffic timer is already started
04:17:13.503923  [04:17:13.495934] [000000A816BAB9CE] [VosMC] wlan: [IH:PMC] pmcEnablePowerSave: 636: Entering pmcEnablePowerSave, power save mode 0
04:17:13.503934  [04:17:13.495993] [000000A816BABE33] [VosMC] wlan: [I :SME] csrRoamCheckForLinkStatusChange: 10217: DEAUTHENTICATION Indication from MAC
04:17:13.503944  [04:17:13.496016] [000000A816BABFE8] [VosMC] wlan: [IH:SME] sme_QosCsrEventInd: 1060: On Session 0 Event 6 received from CSR
04:17:13.503954  [04:17:13.496029] [000000A816BAC0E6] [VosMC] wlan: [IH:SME] sme_QosProcessDisconnectEv: 4910: invoked on session 0
04:17:13.503965  [04:17:13.496047] [000000A816BAC238] [VosMC] wlan: [IH:SME] sme_QosStateTransition: 5886: On session 0 new state=1, old state=0, for AC=0
04:17:13.503976  [04:17:13.496060] [000000A816BAC33A] [VosMC] wlan: [IH:SME] sme_QosStateTransition: 5886: On session 0 new state=1, old state=0, for AC=1
04:17:13.503986  [04:17:13.496073] [000000A816BAC438] [VosMC] wlan: [IH:SME] sme_QosStateTransition: 5886: On session 0 new state=1, old state=0, for AC=2
04:17:13.503997  [04:17:13.496083] [000000A816BAC4F3] [VosMC] wlan: [IH:SME] sme_QosStateTransition: 5886: On session 0 new state=1, old state=0, for AC=3
04:17:13.504008  [04:17:13.496157] [000000A816BACA85] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.504034  [04:17:13.496183] [000000A816BACC65] [VosMC] wlan: [IH:PMC] PMC: entering pmcStopUapsd
04:17:13.504046  [04:17:13.496192] [000000A816BACD1A] [VosMC] wlan: [W :PMC] PMC: Device is already out of UAPSD state. Current state is 1
04:17:13.504057  [04:17:13.496199] [000000A816BACD99] [VosMC] wlan: [IH:SME] sme_QosDeleteBufferedRequests: 6489: Invoked on session 0
04:17:13.504067  [04:17:13.496207] [000000A816BACE35] [VosMC] wlan: [W :SME] sme_QosDeleteBufferedRequests: 6496: Buffered List empty, nothing to delete on session 0
04:17:13.504078  [04:17:13.496215] [000000A816BACECC] [VosMC] wlan: [W :SME] sme_QosDeleteExistingFlows: 6276: Flow List empty, nothing to delete
04:17:13.504089  [04:17:13.496233] [000000A816BAD032] [VosMC] wlan: [IH:SME] sme_QosCsrEventInd: 1121: On Session 0 processed Event 6 with status 0
04:17:13.504099  [04:17:13.496244] [000000A816BAD101] [VosMC] wlan: [W :SME] csrRoamDeregStatisticsReq: List empty, no request from upper layer client(s)
04:17:13.504110  [04:17:13.496256] [000000A816BAD1E0] [VosMC] wlan: [I :SME] csrNeighborRoamIndicateDisconnect: 4493: Disconnect indication on session 0 in state eCSR_NEIGHBOR_ROAM_STATE_CONNECTEDfrom BSSID : 70:3d:15:1a:2b:73
04:17:13.504121  [04:17:13.496286] [000000A816BAD42D] [VosMC] wlan: [ D:SME] csrNeighborRoamIndicateDisconnect: 4574: Neighbor Roam Transition from state eCSR_NEIGHBOR_ROAM_STATE_CONNECTED ==> eCSR_NEIGHBOR_ROAM_STATE_INIT
04:17:13.504131  [04:17:13.496414] [000000A816BADDC4] [VosMC] wlan: [ D:SME] Roam Scan Offload Command 2, Reason 4
04:17:13.504142  [04:17:13.496459] [000000A816BAE128] [VosMC] wlan: [I :SME] csrRoamProcessWmStatusChangeCommand: 11631: session:0, CmdType : 1
04:17:13.504152  [04:17:13.496469] [000000A816BAE1DA] [VosMC] wlan: [W :SME] csrRoamLostLink: 11457: roamInfo.staId (3)
04:17:13.506930  [04:17:13.496476] [000000A816BAE26A] [VosMC] wlan: [I :SME] eCSR_ROAM_RESULT_LOSTLINK 
04:17:13.506967  [04:17:13.496634] [000000A816BAEE3B] [VosMC] wlan: [IH:HDD] CSR Callback: status= 13 result= 7 roamID=0
04:17:13.506978  [04:17:13.497015] [000000A816BB0ADF] [VosMC] wlan: [W :SME] csrScanStartIdleScan: 7568: called
04:17:13.506988  [04:17:13.497027] [000000A816BB0BBD] [VosMC] wlan: [I :SME] csrScanTriggerIdleScan: 7472:   Cannot request IMPS because command pending
04:17:13.506999  [04:17:13.497035] [000000A816BB0C4F] [VosMC] wlan: [I :SME]  csrScanStartIdleScanTimer
04:17:13.507010  [04:17:13.497059] [000000A816BB0E27] [VosMC] wlan: [I :SME] csrScanTriggerIdleScan: 7497: Defer IMPS for 200ms as command processed
04:17:13.507020  [04:17:13.497065] [000000A816BB0E93] [VosMC] wlan: [I :SME]  csrScanStartIdleScanTimer
04:17:13.507031  [04:17:13.497082] [000000A816BB0FD4] [VosMC] wlan: [W :SME] csrTranslateToWNICfgDot11Mode: 2248:   Warning: sees eCSR_CFG_DOT11_MODE_AUTO 
04:17:13.507041  [04:17:13.497157] [000000A816BB157F] [VosMC] wlan: [IL:WDA] =========> WDA_McProcessMsg msgType: 10df 
04:17:13.507051  [04:17:13.497181] [000000A816BB1749] [VosMC] wlan: [I :WDA] ------> WDA_ProcessRoamScanOffloadReq 
04:17:13.507062  [04:17:13.497199] [000000A816BB1896] [VosMC] wlan: [I :WDA] WDA_ConvertSirAuthToWDIAuth: Unknown Auth Type
04:17:13.507073  [04:17:13.497349] [000000A816BB23D8] [VosMC] wlan: [I :PE ] __limProcessSmeDisassocCnf: 2811: received SME_DISASSOC_CNF message
04:17:13.507083  [04:17:13.497503] [000000A816BB2F6A] [VosMC] wlan: [E :PE ] limFTCleanup: Setting psavedsessionEntry= 0000000000000000 to NULL
04:17:13.507095  [04:17:13.498109] [000000A816BB5CFC] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507105  [04:17:13.498652] [000000A816BB859A] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507116  [04:17:13.499119] [000000A816BBA8A2] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507127  [04:17:13.499561] [000000A816BBC9C4] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507149  [04:17:13.500006] [000000A816BBEB31] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507162  [04:17:13.500443] [000000A816BC0BF4] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507173  [04:17:13.500880] [000000A816BC2CAE] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507184  [04:17:13.501311] [000000A816BC4D0B] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507195  [04:17:13.501741] [000000A816BC6D48] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507206  [04:17:13.502164] [000000A816BC8D01] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507216  [04:17:13.502596] [000000A816BCAD61] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507228  [04:17:13.503030] [000000A816BCCDFC] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507239  [04:17:13.503476] [000000A816BCEF68] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507250  [04:17:13.504509] [000000A816BD3CE7] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507262  [04:17:13.504951] [000000A816BD5E05] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.507273  [04:17:13.505384] [000000A816BD7E88] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.568052  [04:17:13.505901] [000000A816BDA549] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.568109  [04:17:13.506620] [000000A816BDDB40] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.568133  [04:17:13.509183] [000000A816BE9B7B] [VosMC] wlan: [E :PE ] limFTCleanup: Setting pftSessionEntry= 0000000000000000 to NULL
04:17:13.568157  [04:17:13.521363] [000000A816C22D01] [VosMC] wlan: [I :PE ] limCleanupRxPath: 670: Cleanup Rx Path for AID : 1psessionEntry->limSmeState : 16, mlmState : 16
04:17:13.568180  [04:17:13.521378] [000000A816C22E11] [VosMC] wlan: [I :PE ] limAdmitControlDeleteSta: 1018: assocId 1 done
04:17:13.568204  [04:17:13.521391] [000000A816C22F04] [VosMC] wlan: [I :PE ] limDeactivateAndChangeTimer: 1545: Deactivated probe after hb timer
04:17:13.568227  [04:17:13.521400] [000000A816C22FB5] [VosMC] wlan: [W :PE ] limDeactivateAndChangeTimer: 1572: Probe after HB timer value is changed = 7
04:17:13.568250  [04:17:13.521427] [000000A816C231C3] [VosMC] wlan: [I :PE ] limDelSta: 2716: Current Operating channel is 1
04:17:13.568273  [04:17:13.521439] [000000A816C232A1] [VosMC] wlan: [I :PE ] limDelSta: 2797: Sessionid 0 :Sending SIR_HAL_DELETE_STA_REQ for STAID: 1 and AssocID: 1 MAC : 70:3d:15:1a:2b:73
04:17:13.568295  [04:17:13.521505] [000000A816C23792] [VosMC] wlan: [I :WDA] <------ WDA_RoamOffloadScanReqCallback 
04:17:13.568318  [04:17:13.521549] [000000A816C23ADC] [VosMC] wlan: [IL:WDA] =========> WDA_McProcessMsg msgType: 102f 
04:17:13.568340  [04:17:13.521561] [000000A816C23BCD] [VosMC] wlan: [I :WDA] ------> WDA_ProcessDelStaReq 
04:17:13.568363  [04:17:13.521615] [000000A816C23FD0] [VosMC] wlan: [I :WDA] <------ WDA_DelSTAReqCallback, wdiStatus: 0
04:17:13.568387  [04:17:13.521636] [000000A816C24164] [VosMC] wlan: [I :PE ] limDeferMsg: 271: Deferred message(0x12B0) limSmeState 1 (prev sme state 1) sysRole 0 mlm state 1 (prev mlm state 1)
04:17:13.568409  [04:17:13.521649] [000000A816C2425C] [VosMC] wlan: [I :SME] Rsp for Roam Scan Offload with reason 4
04:17:13.568452  [04:17:13.525946] [000000A816C384C3] [VosMC] wlan: [I :WDA] <------ WDA_DelSTARspCallback 
04:17:13.568478  [04:17:13.526021] [000000A816C38A4F] [VosMC] wlan: [I :PE ] limProcessStaMlmDelStaRsp: 2472: Del STA RSP received. Status:0 AssocID:1
04:17:13.568809  [04:17:13.526030] [000000A816C38AF8] [VosMC] wlan: [I :PE ] limProcessStaMlmDelStaRsp: 2501: STA AssocID 1 MAC 
04:17:13.568850  [04:17:13.526039] [000000A816C38BA5] [VosMC] wlan: [I :PE ] limPrintMacAddr: 1342: 70:3d:15:1a:2b:73
04:17:13.568876  [04:17:13.526062] [000000A816C38D51] [VosMC] wlan: [W :PE ] limDelBss: 3559: Sessionid 0 : Sending HAL_DELETE_BSS_REQ for bss idx: 0 BSSID:70:3d:15:1a:2b:73
04:17:13.568899  [04:17:13.526074] [000000A816C38E42] [VosMC] wlan: [IL:WDA] =========> WDA_McProcessMsg msgType: 1033 
04:17:13.568922  [04:17:13.526086] [000000A816C38F28] [VosMC] wlan: [I :WDA] ------> WDA_ProcessDelBssReq 
04:17:13.568946  [04:17:13.526134] [000000A816C392C3] [VosMC] wlan: [I :WDA] <------ WDA_DelBSSReqCallback, wdiStatus: 0
04:17:13.568969  [04:17:13.528894] [000000A816C461D5] [VosMC] wlan: [I :WDA] Received WDI_LOST_LINK_PARAMS_IND from WDI 
04:17:13.568991  [04:17:13.528980] [000000A816C46836] [VosMC] wlan: [IH:HDD] CSR Callback: status= 39 result= 0 roamID=0
04:17:13.569014  [04:17:13.528988] [000000A816C468D3] [VosMC] wlan: [I :HDD] hdd_smeRoamCallback : Rssi on Disconnect : -65
04:17:13.569037  [04:17:13.544758] [000000A816C907D2] [VosMC] wlan: [I :WDA] <------ WDA_DelBSSRspCallback 
04:17:13.569061  [04:17:13.544958] [000000A816C9169B] [VosMC] wlan: [W :PE ] limProcessStaMlmDelBssRsp: 2194: STA received the DEL_BSS_RSP for BSSID: 0.
04:17:13.569083  [04:17:13.544998] [000000A816C9199A] [VosMC] wlan: [I :PE ] limProcessStaMlmDelBssRsp: 2216: STA AssocID 1 MAC 
04:17:13.569106  [04:17:13.545018] [000000A816C91B0B] [VosMC] wlan: [I :PE ] limPrintMacAddr: 1342: 70:3d:15:1a:2b:73
04:17:13.569131  [04:17:13.545061] [000000A816C91E54] [VosMC] wlan: [W :PE ] limDeleteDphHashEntry: 3271: Deleting DPH Hash entry for STAID: 1
04:17:13.569152   
04:17:13.569174  [04:17:13.545079] [000000A816C91FAE] [VosMC] wlan: [I :PE ] dphDeleteHashEntry: 403: assocId 1 index 9 STA addr
04:17:13.569197  [04:17:13.545097] [000000A816C920FE] [VosMC] wlan: [I :PE ] dphPrintMacAddr: 481: MAC ADDR = 70:3d:15:1a:2b:73
04:17:13.575692  [04:17:13.545123] [000000A816C922F1] [VosMC] wlan: [I :PE ] limSendDelStaCnf: 795: Sessionid: 0 staDsAssocId: 1 Trigger: 6 statusCode: 0 staDsAddr: 70:3d:15:1a:2b:73
04:17:13.575736  [04:17:13.545174] [000000A816C926CB] [VosMC] wlan: [W :PE ] limSendDelStaCnf: 864: Lim Posting DEAUTH_CNF to Sme.Trigger: 6
04:17:13.575759  [04:17:13.545194] [000000A816C92843] [VosMC] wlan: [I :PE ] limProcessMlmDeauthCnf: 1607: *** Deauthenticated with BSS ***
04:17:13.575784  [04:17:13.545228] [000000A816C92AD6] [VosMC] wlan: [I :PE ] limSendSmeDeauthNtf: 1832: send  eWNI_SME_DISCONNECT_DONE_IND withretCode: 505
04:17:13.575808  [04:17:13.545251] [000000A816C92C83] [VosMC] wlan: [W :PE ] peDeleteSession: 366: Trying to delete a session 0 Opmode 0 BssIdx 0 BSSID: 70:3d:15:1a:2b:73
04:17:13.575832  [04:17:13.545597] [000000A816C94685] [VosMC] wlan: [W :PE ] peDeleteSession: 366: Trying to delete a session 0 Opmode 0 BssIdx 0 BSSID: 70:3d:15:1a:2b:73
04:17:13.575854  [04:17:13.545683] [000000A816C94CEE] [VosMC] wlan: [IL:WDA] =========> WDA_McProcessMsg msgType: 106d 
04:17:13.575876  [04:17:13.545714] [000000A816C94F42] [VosMC] wlan: [I :WDA] ------> WDA_ProcessSetLinkState 
04:17:13.576025  [04:17:13.545874] [000000A816C95B48] [VosMC] wlan: [I :SME] csrRoamCheckForLinkStatusChange: 10270: eWNI_SME_DISCONNECT_DONE_IND RC:0
04:17:13.576037  [04:17:13.545894] [000000A816C95CD0] [VosMC] wlan: [IH:HDD] CSR Callback: status= 12 result= 6 roamID=0
04:17:13.576048  [04:17:13.545909] [000000A816C95DE4] [VosMC] wlan: [I :HDD] ****eCSR_ROAM_DISASSOCIATED****
04:17:13.576060  [04:17:13.545924] [000000A816C95F03] [VosMC] wlan: [I :SME] sme_resetCoexEevent: 13781: isCoexScoIndSet: 0
04:17:13.576070  [04:17:13.545938] [000000A816C9600A] [VosMC] wlan: [I :HDD] hdd_DisConnectHandler: 1640: Disabling queues
04:17:13.576082  [04:17:13.546004] [000000A816C96510] [VosMC] wlan: [I :HDD] hdd_DisConnectHandler: 1679: Set HDD connState to eConnectionState_Disconnecting from 2 
04:17:13.576107  [04:17:13.546019] [000000A816C9662A] [VosMC] wlan: [I :HDD] hdd_connSetConnectionState: 194: ConnectionState Changed from oldState:2 to State:5
04:17:13.576120  [04:17:13.546034] [000000A816C9674B] [VosMC] wlan: [I :HDD] wlan_hdd_decr_active_session: 14350: No.# of active sessions for mode 0 = 0
04:17:13.576131  [04:17:13.546381] [000000A816C98160] [VosMC] wlan: [IL:HDD] hdd_wmm_init: Entered
04:17:13.576141  [04:17:13.546402] [000000A816C982DB] [VosMC] wlan: [E :HDD] wlan: disconnected
04:17:13.576152  [04:17:13.554380] [000000A816CBD976] [VosMC] wlan: [E :HDD] wlan(0) 00:00:00:00:00:00 Standalone
04:17:13.577108  [04:17:13.563788] [000000A816CE9B0F] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
04:17:13.577131  [04:17:13.564196] [000000A816CEB974] [VosMC] wlan: [IL:HDD] WiFi unassociated; gAmpChannel 0 gWiFiChannel 1
04:17:13.577142  [04:17:13.564559] [000000A816CED4C0] [VosMC] wlan: [IH:HDD] hdd_DisConnectHandler: sent disconnected event to nl80211
```

主要几条
```
04:17:13.397171 [04:17:13.385539] [000000A8169A624C] [VosMC] wlan: [W :PE ] limHandleHeartBeatFailure: 518: Heartbeat Failure
04:17:13.397182 [04:17:13.385559] [000000A8169A63AB] [VosMC] wlan: [W :PE ] limHandleHeartBeatFailure: 536: Heart Beat missed from AP. Sending Probe Req
04:17:13.569014 [04:17:13.528988] [000000A816C468D3] [VosMC] wlan: [I :HDD] hdd_smeRoamCallback : Rssi on Disconnect : -65
04:17:13.577142 [04:17:13.564559] [000000A816CED4C0] [VosMC] wlan: [IH:HDD] hdd_DisConnectHandler: sent disconnected event to nl80211
```