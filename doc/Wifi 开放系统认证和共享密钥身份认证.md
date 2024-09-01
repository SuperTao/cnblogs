```
记录开放系统认证和共享密钥认证的区别。

开放系统身份认证(open-systern authentication)
是802.11 要求必备的惟一方式。
由行动式工作站所发出的第一个帧被归类为authentication（身份认证）的管理信息。
802.11 在规格中并未正式将此帧视为身份认证要求（authentication quest）。不过实际上的
作用即是如此。在802.11 中，工作站是以MAC 地址为身份证明。和Ethernet 网络一样，网络上
的MAC 地址必须独一无二，因此可作为工作站的身份证明。基站以这些帧的来源地址作为发送者
的身份证明，此外，并没有以该帧其他字段作为身份证明之用。
身份认证要求包含两个信息元素。首先，身份认证算法代号（Authentication gorithm
Identification）被设置为0，代表使用开放系统认证方式。其次，身份认证交易顺序编号
(Authentication Transaction Sequence number）被设置为1，代表该帧实际上为交易顺序中
第一个帧。
基站接着会处理身份认证要求，然后传回结果。和第一个帧一样，回应帧亦是该类型为
authentication 的管理帧。其中包含三个信息元素：身份认证算法代号 位被设置为0，代表使
用开放系统身份认证，顺序编号为2，另外还有一个状态码（status Code）用来显示身份认证
要求的结果。
开放系统认证常用的加密方式有WPA，WPA2等。

共享密钥身份认证(shared-key authentication）
必须使用WEP
因此只能用于实现了WEP的产品上，虽然目前已经很难找到不支持WEP 的产品。正如其名，「共享密钥身份认证」要求在
进行身份认证之前，必须传递共享密钥给工作站。共享密钥身份认证的理论基础是，如能成功回
应传给它的挑战信息，就证明工作站拥有共享密钥。「共享密钥身份认证」交换程序，使用到了
四个被归类为authentication（身份认证）的管理帧。

wpa_supplicant.conf中的区别。
auth_alg=OPEN 		// 开放系统认证
auth_alog=SHARED 	// 共享密钥认证
auth_alg=OPEN SHARED 	// 开放系统认证，共享密钥认证都支持。
```
LiuTao
2018-11-19