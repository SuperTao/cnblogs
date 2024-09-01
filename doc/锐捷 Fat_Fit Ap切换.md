工作中要使用锐捷的AP和AC进行组网。记录一下RG-AP220-E配置成瘦AP的方法。

使用console口连接，baudrate rate: 9600, 8n1

瘦AP：console密码是ruijie，enable密码是apdebug

胖AP：console密码是admin，没有默认enable密码。

```
User Access Verification

Password:

Ruijie>enable

Password:

Ruijie#ap-m ?
  fat  Fat mode
  fit  Fit mode

Ruijie#ap-m fit
apmode will change to FIT.

Ruijie#show ap-m 
current mode: fit

Ruijie#show ip ?
  arp         Show IP ARP
  interface   IP interface status and configuration
  packet      IP packet
  raw-socket  Show raw socket
  ref         Show Ruijie Express Forward information
  route       IP routing table
  sockets     Show IPv4 sockets
  udp         Show UDP sockets

Ruijie#show ip interface
BVI 1
  IP interface state is: UP
  IP interface type is: BROADCAST
  IP interface MTU is: 1500
  IP address is: 
    192.168.110.1/24 (primary)
  IP address negotiate is: OFF
  Forward direct-broadcast is: OFF
  ICMP mask reply is: ON
  Send ICMP redirect is: ON
  Send ICMP unreachable is: ON
  DHCP relay is: OFF
  Fast switch is: ON
  Help address is: 
  Proxy ARP is: OFF
ARP packet input number: 0
  Request packet     : 0
  Reply packet       : 0
  Unknown packet     : 0
TTL invalid packet number: 0
ICMP packet input number: 0
 Echo request       : 0
 Echo reply         : 0
 Unreachable        : 0
＃退出
Ruijie#end
# 保存
Ruijie>write
```

Tony Liu

2017-12-21