了解一下关于蓝牙Mesh的知识。
蓝牙mesh网络使用，并且依赖于低功耗蓝牙(BLE)。低功耗蓝牙技术是蓝牙mesh使用的无线通信协议栈。
蓝牙BR / EDR能够实现一台设备到另一台设备的连接和通信，建立“一对一”的关系，大多数人所熟悉的“配对”（pairing）一词就是这个意思。
蓝牙mesh能让我们建立无线设备之间的“多对多”（m:m）关系。
此外，设备能够将数据中继到不在初始设备直接无线电覆盖范围内的其他设备。这样，mesh网络就能够跨越非常大的物理区域，并包含大量设备。

蓝牙网络拓扑的基本概念：
1.	节点（Node）
想象一下由数千台设备组成的网络，每台设备均通过低功耗蓝牙（LE）无线连接进行通信。蓝牙mesh网络中的这些设备被称为节点 (node) 。
每个节点都能发送和接收消息。信息能够在节点之间被中继，从而让消息传输至比无线电波正常传输距离更远的位置。
这样的节点网络可以被分布在制造工厂、办公楼、购物中心、商业园区（图2）以及更多环境中。

2.	元素（Elements）
一些节点（如传感器）的电池有可能会被耗尽，而其他节点（如照明设备、制造机械和安防摄像机）则会通过主电网来获取电力。
一些节点的处理能力会高于其他节点。这些节点在mesh网络中可承担更为复杂的任务，扮演不同的角色，表现出以下四个节点特征（Features）：
低功耗 (Low-Power) 特性
功率受限的节点可能会利用低功耗特性来减少无线电接通时间并节省功耗。同时低功耗节点（LPN）可以与friend节点协同工作。
Friend 特性
功率不受限的节点很适合作为friend节点。Friend 节点能够存储发往低功耗节点（LPN）的消息和安全更新；当低功耗节点需要时再将存储的信息传输至低功耗节点。
中继 (Relay) 特性
中继节点能够接收和转发消息，通过消息在节点之间的中继，实现更大规模的网络。节点是否能够具备这一特性取决于其电源和计算能力。
代理 (Proxy) 特性
代理节点能够实现GATT和蓝牙mesh节点之间的mesh消息发送与接收。承担这一角色的节点需要固定的电源和计算资源。

3.	模型 (Model) 和状态 (State)
无论节点位于制造厂房、酒店、办公楼、还是商业园区的网络中，节点的基本功能都由模型 (Model) 来定义和实施。模型位于元素内，元素必须具有至少一个模型。模型能够定义并实施节点的功能和行为，而状态 (State) 能够定义元素的条件。


有些概念介绍比较复杂，Mesh数据传输涉及到数据加密，可以查看下面的链接了解详情。
感觉中继在mesh起比较重要的作用，可以转发数据，有点类似交换机的作用，这样就可以扩大蓝牙数据传输的范围。

相关参考文档：
蓝牙为什么叫蓝牙
https://www.zhihu.com/question/20680730

蓝牙mesh介绍
https://blog.csdn.net/zhanghuaishu0/article/details/78770486