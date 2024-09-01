需要用到锐捷的ac管理2台ap。记录一下。

参考文档

锐捷WLAN无线产品一本通（V6.0）: http://www.ruijie.com.cn/fw/wd/58033


1、确认AC无线交换机和AP是同一个软件版本，使用Ruijie>show verison 查看。

由于两台ap版本不同，一台ap220-E，一台ap220-E(M)，所以从V10升级到V11.4需要的镜像不同。如果都使用同一个镜像，可能会升级失败。

2、确认AP是工作在廋模式下，使用Ruijie>show ap-mode 验证，显示fit是廋模式。

3、有两台ap，一个使用直流电源供电，与ac直连，网口act等亮；另外一台ap通过poe电源与ac相连，act等不良，网速自适应失败。需要手动配置成100M网速才可以连接。

4、 使用tftp升级，防火墙不能关闭，不能使用，添加特例，给tftp权限。

```
Ruijie>enable 
Password:
Ruijie#con
Enter configuration commands, one per line.  End with CNTL/Z.
Ruijie(config)#sh ip int b
Interface                                IP-Address(Pri)      IP-Address(Sec)      Status                 Protocol 
BVI 1                                    192.168.101.4/24     no address           up                     up       
Ruijie(config)#
Ruijie(config)#
Ruijie(config)#
Ruijie(config)#ex
*Jan  1 00:03:18: %SYS-5-CONFIG_I: Configured from console by console
Ruijie#
Ruijie#
Ruijie#ping 3.3.3.3
Sending 5, 100-byte ICMP Echoes to 3.3.3.3, timeout is 2 seconds:
  < press Ctrl+C to break >
.
Success rate is 0 percent (0/1).
Ruijie#

Ruijie#sh int status 
Interface                        Status    Vlan   Duplex   Speed     Type  
-------------------------------- --------  ----   -------  --------- ------
GigabitEthernet 0/1              down      1      Unknown  Unknown   copper
Dot11radio 1/0                   up               AUTO     AUTO      copper
Dot11radio 2/0                   up               AUTO     AUTO      copper
Mobile-Tunnel 1                  up               AUTO     AUTO      copper
Mobile-Tunnel 2                  up               AUTO     AUTO      copper
WBI 1/0                          up               AUTO     AUTO      copper
WBI 1/1                          up               AUTO     AUTO      copper
WBI 1/2                          up               AUTO     AUTO      copper
WBI 1/3                          up               AUTO     AUTO      copper
WBI 1/4                          up               AUTO     AUTO      copper
WBI 2/0                          up               AUTO     AUTO      copper
WBI 2/1                          up               AUTO     AUTO      copper
WBI 2/2                          up               AUTO     AUTO      copper
WBI 2/3                          up               AUTO     AUTO      copper
WBI 2/4                          up               AUTO     AUTO      copper
Ruijie#
Ruijie#
Ruijie#
Ruijie#
Ruijie#con
Ruijie(config)#int gigabitEthernet 0/1

#将ap端口网速设置成100M

Ruijie(config-if-GigabitEthernet 0/1)#speed 100				
Ruijie(config)#sh int st
Ruijie(config)#sh int status 
*Jan  1 00:03:45: %LINK-3-UPDOWN: Interface GigabitEthernet 0/1, changed state to up.
*Jan  1 00:03:45: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet 0/1, changed state to up.
Interface                        Status    Vlan   Duplex   Speed     Type  
-------------------------------- --------  ----   -------  --------- ------
GigabitEthernet 0/1              down      1      Unknown  Unknown   copper
Dot11radio 1/0                   up               AUTO     AUTO      copper
Dot11radio 2/0                   up               AUTO     AUTO      copper
Mobile-Tunnel 1                  up               AUTO     AUTO      copper
Mobile-Tunnel 2                  up               AUTO     AUTO      copper
WBI 1/0                          up               AUTO     AUTO      copper
WBI 1/1                          up               AUTO     AUTO      copper
WBI 1/2                          up               AUTO     AUTO      copper
WBI 1/3                          up               AUTO     AUTO      copper
WBI 1/4                          up               AUTO     AUTO      copper
WBI 2/0                          up               AUTO     AUTO      copper
WBI 2/1                          up               AUTO     AUTO      copper
WBI 2/2                          up               AUTO     AUTO      copper
WBI 2/3                          up               AUTO     AUTO      copper
WBI 2/4                          up               AUTO     AUTO      copper
Ruijie(config)#
Ruijie(config)#ex
*Jan  1 00:03:50: %SYS-5-CONFIG_I: Configured from console by console
Ruijie#
Ruijie#wr
*Jan  1 00:03:51: %CAPWAP-6-STATE_CHANGE: Capwap discovery state changed, from <DISC> to <SELECT>
*Jan  1 00:03:51: %CAPWAP-6-STATE_CHANGE: Capwap discovery state changed, from <SELECT> to <SUCCESS>
*Jan  1 00:03:52: %CAPWAP-6-STATE_CHANGE: (peer - 1) [3.3.3.3] capwap state changed, from <Idle> to <DTLS Setup>
*Jan  1 00:03:52: %CAPWAP-6-STATE_CHANGE: (peer - 1) [3.3.3.3] capwap state changed, from <DTLS Setup> to <Join>
*Jan  1 00:03:52: %CAPWAP-6-STATE_CHANGE: (peer - 1) [3.3.3.3] capwap state changed, from <Join> to <Configure>
*Jan  1 00:03:52: %CAPWAP-6-STATE_CHANGE: (peer - 1) [3.3.3.3] capwap state changed, from <Configure> to <Data Check>
*Jan  1 00:03:52: %CAPWAP-6-STATE_CHANGE: (peer - 1) [3.3.3.3] capwap state changed, from <Data Check> to <Run>
*Jan  1 00:03:52: %CAPWAP-5-PEER_NOTIFY_UP: Peer <3.3.3.3 : 5246 : 1> UP.

Building configuration...
```

Tony Liu

2017-12-26