记录一下四次握手的log。

```
PMK：
PMK（Pairwise Master Key，成对主密钥


STA和AP得到PMK后，将进行密匙派生以得到PTK。最后，PTK被设置到硬件中，
用于数据的加解密。
·由于AP和STA都需要使用PTK，所以二者需要利用EAPOL Key帧进行信息交换。
这就是4-Way Handshake的作用。

由于STA每次关联都需要重新派生这些
Key，所以它们称为PTK（Pairwise Transient Key，成对临时Key），

09-04 16:01:00.393  5955  5955 I wpa_supplicant: wlan0: Associated with 70:3d:15:1a:31:38			// 关联
09-04 16:01:00.393  5955  5955 D wpa_supplicant: wlan0: WPA: Association event - clear replay counter
09-04 16:01:00.393  5955  5955 D wpa_supplicant: wlan0: WPA: Clear old PTK                                    // 清楚原来的PTK，每次连接的PTK都是不一样的。
09-04 16:01:00.393  5955  5955 D wpa_supplicant: TDLS: Remove peers on association
09-04 16:01:00.393  5955  5955 D wpa_supplicant: EAPOL: External notification - portEnabled=0
09-04 16:01:00.393  5955  5955 D wpa_supplicant: EAPOL: SUPP_PAE entering state DISCONNECTED
09-04 16:01:00.393  5955  5955 D wpa_supplicant: EAPOL: Supplicant port status: Unauthorized                // 此时的端口状态，EAPOL状态unauthorized
09-04 16:01:00.393  5955  5955 D wpa_supplicant: nl80211: Set supplicant port unauthorized for 70:3d:15:1a:31:38
09-04 16:01:00.393   960   960 E wificond: Failed to get NL80211_STA_INFO_TX_PACKETS
09-04 16:01:00.393  5955  5955 D wpa_supplicant: EAPOL: SUPP_BE entering state INITIALIZE
09-04 16:01:00.393  5955  5955 D wpa_supplicant: EAPOL: External notification - portValid=0
09-04 16:01:00.393  5955  5955 D wpa_supplicant: EAPOL: External notification - EAP success=0
09-04 16:01:00.393  5955  5955 D wpa_supplicant: EAPOL: External notification - portEnabled=1
09-04 16:01:00.393  5955  5955 D wpa_supplicant: EAPOL: SUPP_PAE entering state CONNECTING
09-04 16:01:00.393  5955  5955 D wpa_supplicant: EAPOL: enable timer tick
09-04 16:01:00.393  5955  5955 D wpa_supplicant: EAPOL: SUPP_BE entering state IDLE
// 设置认证超时时间10s，也就是说四次握手的超时时间是10秒，如果四次握手出错，将会有10秒时间继续握手
// 这10秒内数据连接是有问题的。10s过后才会重新认证，关联。
09-04 16:01:00.393  5955  5955 D wpa_supplicant: wlan0: Setting authentication timeout: 10 sec 0 usec			
09-04 16:01:00.393  1398  1629 E WificondControl: Invalid signal poll result from wificond
09-04 16:01:00.393  5955  5955 D wpa_supplicant: wlan0: Cancelling scan request
09-04 16:01:00.393  5955  5955 D wpa_supplicant: WMM AC: Missing U-APSD configuration
09-04 16:01:00.393  5955  5955 I wpa_supplicant: wlan0: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
// 第一次握手，EAPOL-Key包含AP生成的Nonce(ANonce)
09-04 16:01:00.394  5955  5955 D wpa_supplicant: l2_packet_receive: l2=0x721d027400, wpa_s->l2_wapi=0x721d027380
09-04 16:01:00.394  5955  5955 D wpa_supplicant: l2_packet_receive: src=70:3d:15:1a:31:38 len=121
09-04 16:01:00.394  5955  5955 D wpa_supplicant: wlan0: RX EAPOL from 70:3d:15:1a:31:38				// 第一次握手，收到EAPOL包
09-04 16:01:00.394  5955  5955 D wpa_supplicant: wlan0: Setting authentication timeout: 10 sec 0 usec
09-04 16:01:00.394  5955  5955 D wpa_supplicant: wlan0: IEEE 802.1X RX: version=1 type=3 length=117
09-04 16:01:00.394  5955  5955 D wpa_supplicant: wlan0:   EAPOL-Key type=2
09-04 16:01:00.394  5955  5955 D wpa_supplicant: wlan0:   key_info 0x8a (ver=2 keyidx=0 rsvd=0 Pairwise Ack)
09-04 16:01:00.394  5955  5955 D wpa_supplicant: wlan0:   key_length=16 key_data_length=22
09-04 16:01:00.396  2013  2080 I QCNEJ/CndHalConnector: -> SND notifyRatConnectionStatusChanged(rat = 1, CONNECTED)
09-04 16:01:00.397  5955  5955 D wpa_supplicant:   replay_counter - hexdump(len=8): 00 00 00 00 00 00 00 00
09-04 16:01:00.397  5955  5955 D wpa_supplicant:   key_nonce - hexdump(len=32): 82 19 37 88 e2 a9 51 62 f8 ce e3 98 73 ce 52 e6 32 b8 6a d6 03 9a 48 de df b3 63 51 1a 3a 9c 1e
09-04 16:01:00.397  5955  5955 D wpa_supplicant:   key_iv - hexdump(len=16): 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
09-04 16:01:00.397  5955  5955 D wpa_supplicant:   key_rsc - hexdump(len=8): 00 00 00 00 00 00 00 00
09-04 16:01:00.397  5955  5955 D wpa_supplicant:   key_id (reserved) - hexdump(len=8): 00 00 00 00 00 00 00 00
09-04 16:01:00.397  5955  5955 D wpa_supplicant:   key_mic - hexdump(len=16): 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
09-04 16:01:00.397  5955  5955 D wpa_supplicant: wlan0: State: ASSOCIATED -> 4WAY_HANDSHAKE
09-04 16:01:00.397  5955  5955 D wpa_supplicant: Notifying state change event to hidl control: 4WAY_HANDSHAKE
09-04 16:01:00.399  1398  2072 I WifiService: getWifiApEnabledState uid=1000
09-04 16:01:00.399   994   994 E Parcel  : Reading a NULL string not supported here.
09-04 16:01:00.399  5955  5955 D wpa_supplicant: wlan0: Determining shared radio frequencies (max len 2)
09-04 16:01:00.399   994   994 E Parcel  : Reading a NULL string not supported here.
09-04 16:01:00.400  5955  5955 D wpa_supplicant: wlan0: Shared frequencies (len=1): completed iteration
09-04 16:01:00.400  5955  5955 D wpa_supplicant: wlan0: freq[0]: 2462, flags=0x1
......
09-04 16:01:00.400  5955  5955 D wpa_supplicant: wlan0: WPA: RX message 1 of 4-Way Handshake from 70:3d:15:1a:31:38 (ver=2)            // 提示四次握手的第一个包
09-04 16:01:00.400  5955  5955 D wpa_supplicant: RSN: msg 1/4 key data - hexdump(len=22): dd 14 00 0f ac 04 00 84 a5 48 43 26 dc c2 9e 6c 92 df 8f 28 5d f5
09-04 16:01:00.400  5955  5955 D wpa_supplicant: WPA: PMKID in EAPOL-Key - hexdump(len=22): dd 14 00 0f ac 04 00 84 a5 48 43 26 dc c2 9e 6c 92 df 8f 28 5d f5
09-04 16:01:00.400  5955  5955 D wpa_supplicant: RSN: PMKID from Authenticator - hexdump(len=16): 00 84 a5 48 43 26 dc c2 9e 6c 92 df 8f 28 5d f5
09-04 16:01:00.400  5955  5955 D wpa_supplicant: wlan0: RSN: no matching PMKID found
// 第二次握手
// Supplicant生成SNonce
09-04 16:01:00.401  5955  5955 D wpa_supplicant: WPA: Renewed SNonce - hexdump(len=32): 35 fa 6e 62 22 f8 1a a2 73 e5 00 18 5f 1d 56 93 79 73 f0 3f 55 c6 e3 35 e2 06 82 5d e4 ff 19 ee
09-04 16:01:00.401  5955  5955 D wpa_supplicant: WPA: PTK derivation using PRF(SHA1)
09-04 16:01:00.401  5955  5955 D wpa_supplicant: WPA: PTK derivation - A1=00:0a:f5:32:b1:50 A2=70:3d:15:1a:31:38
09-04 16:01:00.401  5955  5955 D wpa_supplicant: WPA: Nonce1 - hexdump(len=32): 35 fa 6e 62 22 f8 1a a2 73 e5 00 18 5f 1d 56 93 79 73 f0 3f 55 c6 e3 35 e2 06 82 5d e4 ff 19 ee
09-04 16:01:00.401  5955  5955 D wpa_supplicant: WPA: Nonce2 - hexdump(len=32): 82 19 37 88 e2 a9 51 62 f8 ce e3 98 73 ce 52 e6 32 b8 6a d6 03 9a 48 de df b3 63 51 1a 3a 9c 1e
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: PMK - hexdump(len=32): [REMOVED]
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: PTK - hexdump(len=48): [REMOVED]
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: KCK - hexdump(len=16): [REMOVED]
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: KEK - hexdump(len=16): [REMOVED]
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: TK - hexdump(len=16): [REMOVED]
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: WPA IE for msg 2/4 - hexdump(len=22): 30 14 01 00 00 0f ac 04 01 00 00 0f ac 04 01 00 00 0f ac 02 80 00
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: Replay Counter - hexdump(len=8): 00 00 00 00 00 00 00 00
09-04 16:01:00.402  5955  5955 D wpa_supplicant: wlan0: WPA: Sending EAPOL-Key 2/4				// 第二次握手，发送EAPOL
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: Send EAPOL-Key frame to 70:3d:15:1a:31:38 ver=2 mic_len=16 key_mgmt=0x2
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: EAPOL-Key MIC using HMAC-SHA1
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: KCK - hexdump(len=16): [REMOVED]
09-04 16:01:00.402  5955  5955 D wpa_supplicant: WPA: Derived Key MIC - hexdump(len=16): 98 1f 71 89 58 08 ee 9e e6 96 f2 a9 0f f6 4f f0
09-04 16:01:00.402  5955  5955 D wpa_supplicant: RTM_NEWLINK: ifi_index=22 ifname=wlan0 operstate=5 linkmode=1 ifi_family=0 ifi_flags=0x11003 ([UP][LOWER_UP])
09-04 16:01:00.403  1398  1629 D WifiStateMachine:  ConnectedState !SUPPLICANT_STATE_CHANGE_EVENT rt=19506385/19506385 0 0 SSID: eda51_roam_test BSSID: 70:3d:15:1a:31:38 nid: 0 state: FOUR_WAY_HANDSHAKE
09-04 16:01:00.403  1398  1629 D WifiStateMachine:  L2ConnectedState !SUPPLICANT_STATE_CHANGE_EVENT rt=19506386/19506386 0 0 SSID: eda51_roam_test BSSID: 70:3d:15:1a:31:38 nid: 0 state: FOUR_WAY_HANDSHAKE
09-04 16:01:00.403  1398  1629 D WifiStateMachine:  ConnectModeState !SUPPLICANT_STATE_CHANGE_EVENT rt=19506386/19506386 0 0 SSID: eda51_roam_test BSSID: 70:3d:15:1a:31:38 nid: 0 state: FOUR_WAY_HANDSHAKE
09-04 16:01:00.404  1398  1629 D SupplicantStateTracker: CompletedState{ when=0 what=147462 obj= SSID: eda51_roam_test BSSID: 70:3d:15:1a:31:38 nid: 0 state: FOUR_WAY_HANDSHAKE target=com.android.internal.util.StateMachine$SmHandler }

09-04 16:01:00.412  5955  5955 D wpa_supplicant: l2_packet_receive: l2=0x721d027400, wpa_s->l2_wapi=0x721d027380
09-04 16:01:00.413  5955  5955 D wpa_supplicant: l2_packet_receive: src=70:3d:15:1a:31:38 len=155
// 第三次握手， AP派生密钥，发送supplicant,
09-04 16:01:00.413  5955  5955 D wpa_supplicant: wlan0: RX EAPOL from 70:3d:15:1a:31:38				// 第三次握手，收到AP发过来的EAPOL包
09-04 16:01:00.413  5955  5955 D wpa_supplicant: wlan0: IEEE 802.1X RX: version=1 type=3 length=151
09-04 16:01:00.413  5955  5955 D wpa_supplicant: wlan0:   EAPOL-Key type=2
09-04 16:01:00.413  5955  5955 D wpa_supplicant: wlan0:   key_info 0x13ca (ver=2 keyidx=0 rsvd=0 Pairwise Install Ack MIC Secure Encr)
09-04 16:01:00.413  5955  5955 D wpa_supplicant: wlan0:   key_length=16 key_data_length=56
09-04 16:01:00.413  5955  5955 D wpa_supplicant:   replay_counter - hexdump(len=8): 00 00 00 00 00 00 00 01
09-04 16:01:00.413  5955  5955 D wpa_supplicant:   key_nonce - hexdump(len=32): 82 19 37 88 e2 a9 51 62 f8 ce e3 98 73 ce 52 e6 32 b8 6a d6 03 9a 48 de df b3 63 51 1a 3a 9c 1e
09-04 16:01:00.413  5955  5955 D wpa_supplicant:   key_iv - hexdump(len=16): 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
09-04 16:01:00.413  5955  5955 D wpa_supplicant:   key_rsc - hexdump(len=8): 00 00 00 00 00 00 00 00
09-04 16:01:00.413  5955  5955 D wpa_supplicant:   key_id (reserved) - hexdump(len=8): 00 00 00 00 00 00 00 00
09-04 16:01:00.413  5955  5955 D wpa_supplicant:   key_mic - hexdump(len=16): 76 32 3f 2e f5 8d a2 93 d4 48 a3 04 97 31 e9 e9
09-04 16:01:00.413  5955  5955 D wpa_supplicant: WPA: EAPOL-Key MIC using HMAC-SHA1
09-04 16:01:00.413  5955  5955 D wpa_supplicant: RSN: encrypted key data - hexdump(len=56): e4 42 21 3a 98 9f 97 3e bd 07 38 71 17 4f cb 96 78 dd d0 d3 a0 3f 22 02 4a 7b dd 1c d6 95 93 a4 ...
09-04 16:01:00.413  5955  5955 D wpa_supplicant: WPA: Decrypt Key Data using AES-UNWRAP (KEK length 16)
09-04 16:01:00.413  5955  5955 D wpa_supplicant: WPA: decrypted EAPOL-Key key data - hexdump(len=48): [REMOVED]
09-04 16:01:00.413  5955  5955 D wpa_supplicant: wlan0: State: 4WAY_HANDSHAKE -> 4WAY_HANDSHAKE
09-04 16:01:00.413  5955  5955 D wpa_supplicant: wlan0: WPA: RX message 3 of 4-Way Handshake from 70:3d:15:1a:31:38 (ver=2)            // 四次握手第三个包
09-04 16:01:00.413  5955  5955 D wpa_supplicant: WPA: IE KeyData - hexdump(len=48): 30 14 01 00 00 0f ac 04 01 00 00 0f ac 04 01 00 00 0f ac 02 00 00 dd 16 00 0f ac 01 01 00 ae 9e ...
09-04 16:01:00.413  5955  5955 D wpa_supplicant: WPA: RSN IE in EAPOL-Key - hexdump(len=22): 30 14 01 00 00 0f ac 04 01 00 00 0f ac 04 01 00 00 0f ac 02 00 00
09-04 16:01:00.413  5955  5955 D wpa_supplicant: WPA: GTK in EAPOL-Key - hexdump(len=24): [REMOVED]
// 第四次握手
09-04 16:01:00.413  5955  5955 D wpa_supplicant: wlan0: WPA: Sending EAPOL-Key 4/4					// 发送EAPOL包
09-04 16:01:00.413  5955  5955 D wpa_supplicant: WPA: Send EAPOL-Key frame to 70:3d:15:1a:31:38 ver=2 mic_len=16 key_mgmt=0x2
09-04 16:01:00.414  5955  5955 D wpa_supplicant: WPA: EAPOL-Key MIC using HMAC-SHA1
09-04 16:01:00.414  5955  5955 D wpa_supplicant: WPA: KCK - hexdump(len=16): [REMOVED]
09-04 16:01:00.414  5955  5955 D wpa_supplicant: WPA: Derived Key MIC - hexdump(len=16): 42 6f 06 c0 ef b2 ff 4c cd 8e 52 ff 8f 70 a4 c0
09-04 16:01:00.414  5955  5955 D wpa_supplicant: wlan0: WPA: Installing PTK to the driver
09-04 16:01:00.414  5955  5955 D wpa_supplicant: wpa_driver_nl80211_set_key: ifindex=22 (wlan0) alg=3 addr=0x721d0c5a68 key_idx=0 set_tx=1 seq_len=6 key_len=16
09-04 16:01:00.414  5955  5955 D wpa_supplicant: nl80211: KEY_DATA - hexdump(len=16): [REMOVED]
09-04 16:01:00.414  5955  5955 D wpa_supplicant: nl80211: KEY_SEQ - hexdump(len=6): 00 00 00 00 00 00
09-04 16:01:00.414  5955  5955 D wpa_supplicant:    addr=70:3d:15:1a:31:38
09-04 16:01:00.415  5955  5955 D wpa_supplicant: EAPOL: External notification - portValid=1
09-04 16:01:00.415  5955  5955 D wpa_supplicant: wlan0: State: 4WAY_HANDSHAKE -> GROUP_HANDSHAKE
09-04 16:01:00.416  5955  5955 D wpa_supplicant: Notifying state change event to hidl control: GROUP_HANDSHAKE
......
09-04 16:01:00.418  5955  5955 D wpa_supplicant: RSN: received GTK in pairwise handshake - hexdump(len=18): [REMOVED]
09-04 16:01:00.418  5955  5955 D wpa_supplicant: WPA: Group Key - hexdump(len=16): [REMOVED]
09-04 16:01:00.418  5955  5955 D wpa_supplicant: wlan0: WPA: Installing GTK to the driver (keyidx=1 tx=0 len=16)
09-04 16:01:00.418  5955  5955 D wpa_supplicant: WPA: RSC - hexdump(len=6): 00 00 00 00 00 00
09-04 16:01:00.418  1398  1629 D WifiStateMachine:  L2ConnectedState !SUPPLICANT_STATE_CHANGE_EVENT rt=19506401/19506401 0 0 SSID: eda51_roam_test BSSID: 70:3d:15:1a:31:38 nid: 0 state: GROUP_HANDSHAKE
09-04 16:01:00.419  5955  5955 D wpa_supplicant: wpa_driver_nl80211_set_key: ifindex=22 (wlan0) alg=3 addr=0x6075bec231 key_idx=1 set_tx=0 seq_len=6 key_len=16
09-04 16:01:00.419  5955  5955 D wpa_supplicant: nl80211: KEY_DATA - hexdump(len=16): [REMOVED]
09-04 16:01:00.419  5955  5955 D wpa_supplicant: nl80211: KEY_SEQ - hexdump(len=6): 00 00 00 00 00 00
09-04 16:01:00.419  1398  1629 D WifiStateMachine:  ConnectModeState !SUPPLICANT_STATE_CHANGE_EVENT rt=19506401/19506401 0 0 SSID: eda51_roam_test BSSID: 70:3d:15:1a:31:38 nid: 0 state: GROUP_HANDSHAKE
09-04 16:01:00.419  5955  5955 D wpa_supplicant:    broadcast key
09-04 16:01:00.419  1398  1629 D SupplicantStateTracker: CompletedState{ when=0 what=147462 obj= SSID: eda51_roam_test BSSID: 70:3d:15:1a:31:38 nid: 0 state: GROUP_HANDSHAKE target=com.android.internal.util.StateMachine$SmHandler }
09-04 16:01:00.420  5955  5955 I wpa_supplicant: wlan0: WPA: Key negotiation completed with 70:3d:15:1a:31:38 [PTK=CCMP GTK=CCMP]
09-04 16:01:00.420  5955  5955 D wpa_supplicant: wlan0: Cancelling authentication timeout
09-04 16:01:00.421  5955  5955 D wpa_supplicant: wlan0: State: GROUP_HANDSHAKE -> COMPLETED


四次握手：kernel
16:01:00.398896  [16:01:00.383270] [00000057392A15C3] [VosRX] wlan: [I :TL ] This is RX UC frame
16:01:00.399110  [16:01:00.383275] [00000057392A1622] [VosRX] wlan: [IM:TL ] ****Received rate Index = 0 type=2 subtype=8****
16:01:00.399288  [16:01:00.383284] [00000057392A16D3] [VosRX] wlan: [I :TL ] WLAN TL:RX Frame  EAPOL EtherType 34958 - processing
16:01:00.399467  [16:01:00.383435] [00000057392A2229] [VosRX] wlan: [I :HDD] wlan_hdd_log_eapol: 3222: RX: M1 packet                // 第一次握手
16:01:00.399688  [16:01:00.383440] [00000057392A228E] [VosRX] wlan: [I :HDD] STA RX EAPOL                                                            // 接收
16:01:00.399910  [16:01:00.383496] [00000057392A26BD] [VosRX] wlan: [I :VOS] VPKT [1428]: [0000000000000000] Packet returned, type 3[RX_RAW]
16:01:00.400117  [16:01:00.383748] [00000057392A39B8] [VosMC] wlan: [IM:HDD] ULA auth StaId= 3. Changing TL state to CONNECTEDat Join time
16:01:00.400301  [16:01:00.383759] [00000057392A3A7F] [VosMC] wlan: [IL:HDD] hdd_wmm_assoc: Entered
16:01:00.400472  [16:01:00.383764] [00000057392A3ADA] [VosMC] wlan: [IL:HDD] hdd_wmm_assoc: U-APSD mask is 0x00
16:01:00.400642  [16:01:00.383774] [00000057392A3B96] [VosMC] wlan: [I :SME] sme_UpdateDSCPtoUPMapping: QOS Mapping IE not present
16:01:00.400816  [16:01:00.383778] [00000057392A3BEA] [VosMC] wlan: [IL:HDD] hdd_wmm_init: Entered
16:01:00.400991  [16:01:00.383782] [00000057392A3C2F] [VosMC] wlan: [IL:HDD] hdd_wmm_assoc: Exiting
16:01:00.401157  [16:01:00.383787] [00000057392A3C89] [VosMC] wlan: [I :HDD] hdd_AssociationCompletionHandler: 2617: Enabling queues
16:01:00.401310  [16:01:00.383836] [00000057392A4042] [VosMC] wlan: [I :HDD] wlan_hdd_sta_tdls_init: 1086: TDLS INIT DONE set to 1, no point in re-init
16:01:00.401459  [16:01:00.383841] [00000057392A40A6] [VosMC] wlan: [I :HDD] wlan_hdd_tdls_connection_callback, update 2000 discover 20000
16:01:00.401606  [16:01:00.384127] [00000057392A5621] [VosMC] wlan: [W :SME] csrRoamCompletion: 11405:   indicates association completion. roamResult = 0
16:01:00.401751  [16:01:00.384140] [00000057392A5713] [VosMC] wlan: [IH:HDD] CSR Callback: status= 5 result= 0 roamID=0
16:01:00.401902  [16:01:00.384154] [00000057392A5812] [VosMC] wlan: [I :SME] csrRoamProcessResults: 6022: Set full_power_till_set_key to make sure we wait until keys are set before going to BMPS
16:01:00.402059  [16:01:00.384160] [00000057392A588A] [VosMC] wlan: [I :PMC] pmcStartTrafficTimer: 886: Enter. Timer duration is 8000
16:01:00.402213  [16:01:00.384167] [00000057392A5914] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
16:01:00.402369  [16:01:00.384185] [00000057392A5A6A] [VosMC] wlan: [IH:VOS] vos_timer_stop: Timer Addr inside voss_stop : 0x0000000000000000
16:01:00.402524  [16:01:00.384262] [00000057392A602F] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS SYS MC Message queue
16:01:00.402688  [16:01:00.384274] [00000057392A6110] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
16:01:00.402842  [16:01:00.384281] [00000057392A619E] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS WDA MC Message queue
16:01:00.403013  [16:01:00.384287] [00000057392A6213] [VosMC] wlan: [IL:WDA] =========> WDA_McProcessMsg msgType: 1130 
16:01:00.403194  [16:01:00.384340] [00000057392A6608] [VosMC] wlan: [IH:VOS] vos_list_remove_front: list empty
16:01:00.403384  [16:01:00.384347] [00000057392A668C] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS PE MC Message queue
16:01:00.403569  [16:01:00.384359] [00000057392A6770] [VosMC] wlan: [I :SYS] tx_timer_deactivate() called for timer Heartbeat TIMEOUT
16:01:00.403892  [16:01:00.384363] [00000057392A67C4] [VosMC] wlan: [IH:VOS] vos_timer_stop: Timer Addr inside voss_stop : 0x0000000000000000
16:01:00.404048  [16:01:00.384368] [00000057392A6820] [VosMC] wlan: [IH:VOS] vos_timer_stop: Cannot stop timer in state = 19
16:01:00.404207  [16:01:00.384374] [00000057392A6898] [VosMC] wlan: [W :PE ] limDeactivateAndChangeTimer: 1498: Deactivated heartbeat link monitoring
16:01:00.406261  [16:01:00.384381] [00000057392A691A] [VosMC] wlan: [W :PE ] limDeactivateAndChangeTimer: 1530: HeartBeat timer value is changed = 0
16:01:00.406427  [16:01:00.384408] [00000057392A6B2A] [VosMC] wlan: [I :PMC] pmmSendPowerSaveCfg: 799: pmmBmps: Sending WDA_PWR_SAVE_CFG to HAL
16:01:00.406567  [16:01:00.384418] [00000057392A6BE3] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS WDA MC Message queue
16:01:00.406704  [16:01:00.384423] [00000057392A6C45] [VosMC] wlan: [IL:WDA] =========> WDA_McProcessMsg msgType: 105e 
16:01:00.406841  [16:01:00.384429] [00000057392A6CB1] [VosMC] wlan: [I :WDA] ------> WDA_ProcessSetPwrSaveCfgReq 
16:01:00.406975  [16:01:00.384474] [00000057392A7010] [VosMC] wlan: [IH:VOS] Timer Addr inside voss_start : 0x0000000000000000 
16:01:00.407110  [16:01:00.385179] [00000057392AA519] [wpa_s] wlan: [I :HDD] Enter:__wlan_hdd_cfg80211_dump_survey
16:01:00.407304  [16:01:00.385448] [00000057392AB934] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.407450  [16:01:00.386824] [00000057392B2076] [VosMC] wlan: [IH:VOS] vos_timer_stop: Timer Addr inside voss_stop : 0x0000000000000000
16:01:00.407595  [16:01:00.386840] [00000057392B2191] [VosMC] wlan: [I :WDA] <------ WDA_SetPwrSaveCfgReqCallback 
16:01:00.407732  [16:01:00.386875] [00000057392B242F] [VosMC] wlan: [IH:VOS] vos_list_remove_front: list empty
16:01:00.407877  [16:01:00.387576] [00000057392B58DB] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.408018  [16:01:00.392035] [00000057392CA743] [wpa_s] wlan: [I :HDD] Exit:__wlan_hdd_cfg80211_del_key
16:01:00.408163  [16:01:00.392320] [00000057392CBC9F] [wpa_s] wlan: [I :HDD] Exit:__wlan_hdd_cfg80211_del_key
16:01:00.408310  [16:01:00.392899] [00000057392CE814] [wpa_s] wlan: [I :HDD] Exit:__wlan_hdd_cfg80211_del_key
16:01:00.408449  [16:01:00.393270] [00000057392D03E1] [wific] wlan: [I :HDD] Enter:__wlan_hdd_cfg80211_get_station
16:01:00.408675  [16:01:00.393280] [00000057392D0498] [wific] wlan: [I :HDD] __wlan_hdd_cfg80211_get_station: Roaming in progress, so unable to proceed this request
16:01:00.408813  [16:01:00.393448] [00000057392D113A] [wpa_s] wlan: [I :HDD] Enter:__wlan_hdd_change_station
16:01:00.408960  [16:01:00.393458] [00000057392D11E5] [wpa_s] wlan: [I :HDD] Exit:__wlan_hdd_change_station
16:01:00.409098  [16:01:00.394157] [00000057392D4667] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.409312  [16:01:00.394741] [00000057392D7246] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.409470  [16:01:00.395345] [00000057392D9F98] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.409630  [16:01:00.395958] [00000057392DCD82] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.409839  [16:01:00.396674] [00000057392E0331] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.410002  [16:01:00.402372] [00000057392FAE93] [wpa_s] wlan: [IL:HDD] hdd_wmm_select_queue: up=6 QIndex:4
16:01:00.410166  [16:01:00.402385] [00000057392FAF68] [wpa_s] wlan: [IL:HDD] hdd_wmm_select_queue: up=6 QIndex:4
16:01:00.410313  [16:01:00.402415] [00000057392FB1AD] [wpa_s] wlan: [I :HDD] __hdd_hard_start_xmit It must be a Eapol/Wapi/DHCP packet device_mode:0
16:01:00.410526  [16:01:00.402424] [00000057392FB250] [wpa_s] wlan: [I :TL ] WLAN TL:Packet pending indication for STA: 3 AC: 4
16:01:00.410674  [16:01:00.402431] [00000057392FB2E4] [wpa_s] wlan: [IH:TL ] WLAN TL:Packet pending indication for STA: 3 AC: 4 State: 1
16:01:00.410842  [16:01:00.402436] [00000057392FB33B] [wpa_s] wlan: [IH:TL ] Serializing WDA TX Start Xmit event
16:01:00.410996  [16:01:00.402786] [00000057392FCD92] [VosTX] wlan: [I :VOS] VosTXThread: Servicing the VOS TL TX Message queue
16:01:00.411224  [16:01:00.402806] [00000057392FCEFF] [VosTX] wlan: [I :TL ] WLAN TL: WLANTL_STATxConn fetching packet from HDD for AC: 4 AC Mask: 0 Pkt Pending: 0
16:01:00.411384  [16:01:00.402817] [00000057392FCFCB] [VosTX] wlan: [I :VOS] VPKT [963]: [0000000000000000] Packet allocated, type TX_802_3_DATA

16:01:00.416780  [16:01:00.403016] [00000057392FDECE] [VosTX] wlan: [I :HDD] wlan_hdd_log_eapol: 3227: TX: M2 packet        // 第二个包
16:01:00.416951  [16:01:00.403042] [00000057392FE0BC] [VosTX] wlan: [I :HDD] STA TX EAPOL                                                    // 发送
16:01:00.417105  [16:01:00.403047] [00000057392FE114] [VosTX] wlan: [I :HDD] hdd_tx_fetch_packet_cbk :STA TX ARP Received in TL 
16:01:00.417602  [16:01:00.403072] [00000057392FE2F1] [VosTX] wlan: [IL:TL ] WLAN TL: Dump TX meta info: txFlags:4, qosEnabled:1, ac:6, isEapol:1, fdisableFrmXlt:1, frmType:2
16:01:00.417816  [16:01:00.403078] [00000057392FE367] [VosTX] wlan: [IH:TL ] WLAN TL:GetFrames STA: 3 EAPOLPktPending 0
16:01:00.418066  [16:01:00.403084] [00000057392FE3DE] [VosTX] wlan: [I :TL ] WLAN TL: WLANTL_STATxConn fetching packet from HDD for AC: 4 AC Mask: 0 Pkt Pending: 0
16:01:00.418268  [16:01:00.403090] [00000057392FE448] [VosTX] wlan: [I :TL ] WLAN TL:No more data at HDD status 16
16:01:00.418439  [16:01:00.403094] [00000057392FE49A] [VosTX] wlan: [I :TL ] WLAN TL: WLANTL_STATxConn no more packets in HDD for AC: 4 AC Mask: 0
16:01:00.418615  [16:01:00.403123] [00000057392FE6C0] [VosTX] wlan: [I :HDD] WDTS_TxPacket :Transmitting ARP packet
16:01:00.418793  [16:01:00.403174] [00000057392FEA93] [VosTX] wlan: [IH:TL ] WLAN TL:No station registered with TL at this point
16:01:00.418970  [16:01:00.403188] [00000057392FEB9D] [VosTX] wlan: [I :WDI] WDTS_TxPacketComplete: Management frame Tx complete status: 0
16:01:00.419159  [16:01:00.403211] [00000057392FED62] [VosTX] wlan: [I :VOS] VPKT [1428]: [0000000000000000] Packet returned, type 2[TX_802_3_DATA]
16:01:00.419345  [16:01:00.404711] [0000005739305DF2] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.419551  [16:01:00.405222] [000000573930844A] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.419756  [16:01:00.405772] [000000573930AD8D] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.419958  [16:01:00.411968] [0000005739327E42] [VosRX] wlan: [I :VOS] VPKT [779]: [0000000000000000] Packet allocated, type 3[RX_RAW]
16:01:00.420156  [16:01:00.412004] [00000057393280D7] [VosRX] wlan: [I :VOS] VPKT [779]: [0000000000000000] Packet allocated, type 3[RX_RAW]
16:01:00.420325  [16:01:00.412035] [0000005739328324] [VosRX] wlan: [I :TL ] Management Frame, not need to handle with Stat
16:01:00.420702  [16:01:00.412080] [0000005739328694] [VosRX] wlan: [I :TL ] This is RX UC frame
16:01:00.421034  [16:01:00.412085] [00000057393286F5] [VosRX] wlan: [IM:TL ] ****Received rate Index = 0 type=2 subtype=8****
16:01:00.421304  [16:01:00.412092] [0000005739328769] [VosRX] wlan: [I :TL ] Current new Data RSSI is -44, averaged Data RSSI is -53
16:01:00.421561  [16:01:00.412100] [0000005739328809] [VosRX] wlan: [I :TL ] WLAN TL:RX Frame  EAPOL EtherType 34958 - processing

16:01:00.421831  [16:01:00.412276] [000000573932953B] [VosRX] wlan: [I :HDD] wlan_hdd_log_eapol: 3234: RX: M3 packet            // 第三个包
16:01:00.422151  [16:01:00.412281] [000000573932959F] [VosRX] wlan: [I :HDD] STA RX EAPOL                                                    // 发送
16:01:00.422352  [16:01:00.412291] [0000005739329662] [VosMC] wlan: [I :VOS] VosMCThread: Servicing the VOS PE MC Message queue
16:01:00.422529  [16:01:00.412314] [0000005739329811] [VosMC] wlan: [I :VOS] vos_list_peek_next: list empty
16:01:00.422812  [16:01:00.412346] [0000005739329A7C] [VosRX] wlan: [I :VOS] VPKT [1428]: [0000000000000000] Packet returned, type 3[RX_RAW]
16:01:00.423016  [16:01:00.412454] [000000573932A29A] [VosMC] wlan: [I :PE ] limEncTypeMatched: 1815: Beacon/Probe:: Privacy :1 WPA Present:1 RSN Present: 1
16:01:00.423216  [16:01:00.412460] [000000573932A315] [VosMC] wlan: [I :PE ] limEncTypeMatched: 1819: pSession:: Privacy :1 EncyptionType: 4 WPS 0 OSEN 0
16:01:00.423552  [16:01:00.412497] [000000573932A5CE] [VosMC] wlan: [I :VOS] VPKT [1428]: [0000000000000000] Packet returned, type 3[RX_RAW]
16:01:00.423805  [16:01:00.413233] [000000573932DD0D] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.424046  [16:01:00.413661] [000000573932FD28] [cnss_] wlan: [I :HDD] __wlan_hdd_cfg80211_get_wifi_info: 6759: Rcvd req for FW version FW version is CNSS-PR-4-0-509-158621-1
16:01:00.424261  [16:01:00.414148] [00000057393321A7] [wpa_s] wlan: [IL:HDD] hdd_wmm_select_queue: up=6 QIndex:4
16:01:00.424521  [16:01:00.414156] [0000005739332240] [wpa_s] wlan: [IL:HDD] hdd_wmm_select_queue: up=6 QIndex:4
16:01:00.428462  [16:01:00.414185] [0000005739332466] [wpa_s] wlan: [I :HDD] __hdd_hard_start_xmit It must be a Eapol/Wapi/DHCP packet device_mode:0
16:01:00.428803  [16:01:00.414212] [000000573933266B] [wpa_s] wlan: [I :TL ] WLAN TL:Packet pending indication for STA: 3 AC: 4
16:01:00.429820  [16:01:00.414218] [00000057393326EA] [wpa_s] wlan: [IH:TL ] WLAN TL:Packet pending indication for STA: 3 AC: 4 State: 1
16:01:00.430116  [16:01:00.414223] [000000573933274C] [wpa_s] wlan: [IH:TL ] Serializing WDA TX Start Xmit event
16:01:00.430331  [16:01:00.414518] [0000005739333D62] [VosTX] wlan: [I :VOS] VosTXThread: Servicing the VOS TL TX Message queue
16:01:00.430528  [16:01:00.414533] [0000005739333E89] [VosTX] wlan: [I :TL ] WLAN TL: WLANTL_STATxConn fetching packet from HDD for AC: 4 AC Mask: 0 Pkt Pending: 0
16:01:00.430708  [16:01:00.414543] [0000005739333F49] [VosTX] wlan: [I :VOS] VPKT [963]: [0000000000000000] Packet allocated, type TX_802_3_DATA
16:01:00.430870  [16:01:00.414602] [00000057393343AE] [wpa_s] wlan: [I :HDD] Enter:__wlan_hdd_cfg80211_add_key
16:01:00.431038  [16:01:00.414610] [000000573933444F] [wpa_s] wlan: [I :HDD] __wlan_hdd_cfg80211_add_key: device_mode = WLAN_HDD_INFRA_STATION (0)
16:01:00.431194  [16:01:00.414616] [00000057393344C1] [wpa_s] wlan: [I :HDD] __wlan_hdd_cfg80211_add_key: called with key index = 0 & key length 16 & seq length 6
16:01:00.431350  [16:01:00.414621] [0000005739334516] [wpa_s] wlan: [IM:HDD] __wlan_hdd_cfg80211_add_key: encryption type 6
16:01:00.431504  [16:01:00.414626] [0000005739334580] [wpa_s] wlan: [I :HDD] __wlan_hdd_cfg80211_add_key- 13077: setting pairwise key
16:01:00.431664  [16:01:00.414634] [0000005739334610] [wpa_s] wlan: [IM:HDD] __wlan_hdd_cfg80211_add_key: set key for peerMac 70:3d:15:1a:31:38, direction 2
16:01:00.431829  [16:01:00.414640] [0000005739334690] [wpa_s] wlan: [IH:TL ] Serializing WDA_DS_FinishULA event

16:01:00.432003  [16:01:00.414710] [0000005739334BCD] [VosTX] wlan: [I :HDD] wlan_hdd_log_eapol: 3240: TX: M4 packet            // 第四个包
16:01:00.432188  [16:01:00.414716] [0000005739334C46] [VosTX] wlan: [I :HDD] STA TX EAPOL
16:01:00.432380  [16:01:00.414721] [0000005739334C98] [VosTX] wlan: [I :HDD] hdd_tx_fetch_packet_cbk :STA TX ARP Received in TL 
16:01:00.432583  [16:01:00.414738] [0000005739334DE5] [VosTX] wlan: [IL:TL ] WLAN TL: Dump TX meta info: txFlags:4, qosEnabled:1, ac:6, isEapol:1, fdisableFrmXlt:1, frmType:2
16:01:00.432768  [16:01:00.414745] [0000005739334E65] [VosTX] wlan: [IH:TL ] WLAN TL:GetFrames STA: 3 EAPOLPktPending 0
16:01:00.432942  [16:01:00.414752] [0000005739334EF7] [VosTX] wlan: [I :TL ] WLAN TL: WLANTL_STATxConn fetching packet from HDD for AC: 4 AC Mask: 0 Pkt Pending: 0
16:01:00.433113  [16:01:00.414757] [0000005739334F49] [VosTX] wlan: [I :TL ] WLAN TL:No more data at HDD status 16
16:01:00.433278  [16:01:00.414761] [0000005739334FA3] [VosTX] wlan: [I :TL ] WLAN TL: WLANTL_STATxConn no more packets in HDD for AC: 4 AC Mask: 0
16:01:00.433440  [16:01:00.414797] [000000573933525C] [VosTX] wlan: [I :HDD] WDTS_TxPacket :Transmitting ARP packet
16:01:00.433602  [16:01:00.414860] [0000005739335708] [VosTX] wlan: [IH:TL ] WLAN TL:No station registered with TL at this point
16:01:00.433765  [16:01:00.414882] [00000057393358AA] [VosTX] wlan: [I :WDI] WDTS_TxPacketComplete: Management frame Tx complete status: 0
```