![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211001143701529-845113888.png)

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211002154419814-1486927816.png)


Beacon Timestamp:
Timestamp（时戳）位，可用来同步BSS 中的工作站BSS 的主计时器会
定期发送目前已作用的微秒数。当计数器到达最大值时，便会从头开始计数。（对一个长度64bit、可计数超过580,000 年的计数器而言，很难会遇到有从头开始计数的一天。

Beacon Interval:
每隔一段时间就会发出一个Beacon（信标）信号用来宣布802.11网络的存在。Beacon帧中除了包含BSS参数的信息，也包含接入点缓存帧的信息，因此移动式工作站要仔细聆听Beacon信号。此帧长度为16位，用来设定Beacon信号之间相隔多少时间单位。时间单位通常缩写为TU，代表1024微秒(microsecond)，相当于1毫秒(millisecond)。Beacon通常会被设定为100个时间单位，相当于每100毫秒，也就是0.1秒传送一次Beacon信号。
Capability Info

AP支持的速率
![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211001144341949-1526858773.png)

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211001144355810-803938476.png)


Starting Chaannel: 第一信道编号即是符合功率限制的最低信道
Number of Channels: 符合功率限制的频段大小，是由信道数来指定。信道大小随PHY和国家码的不同而有所不同。
Max Tx Power: 最大传输功率，以dBm 为单位

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211001144406146-967693841.png)

TIM: 数据待传信息，AP会为处于休睡状态的工作站暂存帧。每隔一段时间，AP就会尝试传递这些暂存帧给休眠中的工作站。如此安排的理由是，启动发送器比启动接收器所耗费的电力还要多。802.11的设计者预见未来将会有以电池供电的移动工作站；定期发送暂存帧给工作站的这个决定，主要是为了延长设备的电池使用时间。将TIM（数据待传指示信息）信息元素送到网络上，指示有哪些工作站需要接收待传数据，只是此过程的一部分。
DTIM Count：此位的长度为一个字节，代表下一个DTIM（数据待传指示传递信息）帧发送前，即将发送的Beacon 帧数。
DTIM Period: 此位的长度为一个字节，代表两个DTIM 帧之间的Beacon interval 数。0 值目前保留未用。DTIM 会由此期间倒数至0。
Bitmap Control: 此位可进一步划分为两个次位。Bit 0 用来表示连接识别码0 的待传状态，主要是保留给组播使用。其他七个bit 则是保留给Bitmap Offset（bit对映偏移）次位使用。为了节省频宽，可以通过Bitmap Offset 次位，只发送一部分的虚拟bit 对映。BitmapOffset 是相对于虚拟bit 对映的开头处。利用Bitmap Offset 次位及Length 位，802.11工作站可以推断虚拟bit 对映有哪些部分包括在内。

加密的相关信息

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211002154115154-35033141.png)


Station Count: 有多少台STA与AP相连

![](https://img2020.cnblogs.com/blog/745188/202110/745188-20211001144430555-2138323914.png)