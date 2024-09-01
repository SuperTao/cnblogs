#### probe request

主动扫描通过发送probe request帧进行

STA会在每个信道上发送probe request进行扫描。

probe request会向广播地址FF:FF:FF:FF:FF:FF发送。

发送的STA可以指定SSID，只有指定的AP才会进行回复。

如果probe request的SSID是0，所有的AP都会恢复probe response.

* 不带SSID的扫描

SSID为0，所有的设备都会恢复probe response

![](https://img2020.cnblogs.com/blog/745188/202109/745188-20210925155620725-1397609355.png)

* 带SSID的扫描probe request

如下,只有特定的设备会回复。

![](https://img2020.cnblogs.com/blog/745188/202109/745188-20210925155632046-1640036725.png)

#### Probe response

![](https://img2020.cnblogs.com/blog/745188/202109/745188-20210925155645804-2103746645.png)

#### 连接之前的probe request

连接之前也会发出probe request,其中的目的地址是确定的MAC

![](https://img2020.cnblogs.com/blog/745188/202109/745188-20210925155708793-512609731.png)

接收到probe request之后，对应的AP也会恢复probe response

```
在主动扫描中，工作站扮演比较积极的角色。在每个频道上，工作站 都会岭出Probe Request 帧，请求某个特定网络予以回
应。主动扫描系主动试图寻找网络，而不 是听候网络宣告本身的存在。使用主动扫描的工作站将会以如下的程序扫描频道表
所列的频道：
1.跳至某个频道，然后等候来讯显示，或者等到ProbeDelay 计时器逾时。如果在这个频道收得到帧，就证明该频道有人使
用，因此可以加以探 测。此计时器用来防止某个闲置频道让整个程序停摆；工作站不会一直听候帧到来。
2.利用基本的DCF 访问程序取得介质使用权，然后送出一个Probe Request 帧。
3.至少等候一段最短的频道时间（即MinChannelTime）。
a.如果介质并不忙碌，表示没有网络存在。因此可以跳至下个频道。
b.如果在MinChannelTime 这段期间介质非常忙碌，就继续等候一段时间，直 到 最长的频道时间（即MaxChannelTime），
然后处理任何的Probe Response 帧。
当网络收到搜寻其所属之延伸服务组合的Probe Request（探查要求），就会发出Probe Response（探查回应）帧。为了在
舞会中找到朋友，各位或许会绕著舞池大声叫喊对方的名字。 （虽然这并不礼貌，不过如果真想找到朋友，大概没有其他选
择。）如果对方听见了，她就会出 声回应，至于其他人根本就不会理你（希望如此）。Probe Request 框的作用类似，不过
在Probe Request 帧当中可以使用broadcast SSID，如此一来，该区所有的802.11 网络都会以Probe Response 加以回应。
```
