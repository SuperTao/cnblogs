### 参考链接

http://pavelhan.tech/post/2021-01-12-00-WiFi%E4%B8%AD%E7%9A%84BeaconTIM%E4%B8%8EDTIM%E6%A6%82%E5%BF%B5%E6%80%BB%E7%BB%93/

https://www.cnblogs.com/god-of-death/p/8098643.html

https://blogs.arubanetworks.com/industries/802-11-tim-and-dtim-information-elements/

https://dot11ap.wordpress.com/timdtimatim/

#### TIM和DTIM

TIM：每一个Beacon的帧中都有一个TIM 信息元素 ，它主要用来由AP通告它管辖下的哪个STA有信息现在缓存在AP中，而在TIM中包含一个 Bitmap control 字段，它最大是251个字节，每一位映射一个STA，当为1时表示该位对应的STA有信息在AP中。总之，收到 与自己关联的TIM就要发送PS-POLL帧来与AP取来联系并取得它的缓存帧了。标准的TIM中仅仅指示AP缓存的单播信息。

DTIM：这个是TIM的特殊情况，当发送几个TIM之后，就要发送一个DTIM，其除了缓存单播信息，也同时指示AP缓存的组播或广播信息，一旦AP发送了DTIM, STA就必须处于清醒，因为广播或组播无重发机制，不醒来数据就收不到了。

**也就是说DTIM里面会指示是否有组播数据，也会指示是否有单播数据。**

**如果beacon包含的DTIM里面有组播数据，也包含了单播的数据。STA会马上唤醒，接受组播数据，接受完成之后，会继续接收单播数据。**

### 功耗优化

需要根据DTIM Period来设置。

```
DTIM Count - This field indicates how many beacon frames till the next DTIM.

A DTIM count field of 0 indicates that TIM is a DTIM.

A DTIM count field of 1 indicates the next beacon is a DTIM.

 
DTIM Period - This field indicates the beacon intervals till a DTIM.

A DTIM period field of 1 indicates every other beacon is a DTIM.

A DTIM period field of 3 indicates every third beacon is a DTIM.

A DTIM period field of 5 indicates every fifth beacon is a DTIM.
```

STA通过listen interval来设置监听beacon的间隔。可以设置成和DTIM的间隔一样，或者是DTIM Period的倍数。

这样一来，每次DTIM到来的时候，既可以接收广播数据，也可以接收单播的数据。

* 举例说明：

STA里面都会有一个关于接收DTIM间隔的最大值。

例如STA最大值监听间隔是5，而路由器的DTIM period是1。那么刚好可以每5个beacon醒来一次接收数据。

例如STA最大值监听间隔是5，而路由器的DTIM period是3。那么STA为了同步，就必须每3个beacon醒来一次接受数据。这样功耗就高了一些。

例如STA最大值监听间隔是10，而路由器的DTIM period是3。那么STA为了同步，就必须每9个beacon醒来一次接受数据。功耗就降低了。