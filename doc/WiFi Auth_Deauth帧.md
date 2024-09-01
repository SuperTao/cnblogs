https://www.cnblogs.com/helloworldtoyou/p/9981900.html

认证阶段是建立连接的第一步，不过没有加密数据在这个阶段。

身份认证阶段不需要考虑安全问题，因为在这个阶段只是 AP 要确认 Station 是不是 802.11 设备，确认彼此可以正常通讯。

802.11 auth请求，开放系统认证，序号为1

![](https://img2020.cnblogs.com/blog/745188/202109/745188-20210925173101533-7128653.png)

802.11 Auth回复，序号为2

![](https://img2020.cnblogs.com/blog/745188/202109/745188-20210925173114544-1298753482.png)

Deauth

![](https://img2020.cnblogs.com/blog/745188/202109/745188-20210925173201385-1519524187.png)

Deauthentication Reason Code Table

https://www.cisco.com/assets/sol/sb/WAP371_Emulators/WAP371_Emulator_v1-0-1-5/help/Apx_ReasonCodes2.html