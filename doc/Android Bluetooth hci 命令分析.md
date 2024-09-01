Android在连接BLE设备的时候，遇到连接没多久就自动断开的情况。通过HCI来分析一下。

```
BLE设备发送连接参数更新请求
3909	15:53:01.224737	TexasIns_f0:d3:41 (Hon-RFID3)	HandHeld_e0:e5:4f (EDA)	L2CAP	21	Rcvd Connection Parameter Update Request
回复BLE发送过来的请求
3910	15:53:01.225744	HandHeld_e0:e5:4f (EDA)	TexasIns_f0:d3:41 (Hon-RFID3)	L2CAP	15	Sent Connection Parameter Update Response (Accepted)
发送连接更新请求，host通过hci发给controller
3911	15:53:01.227044	host	controller	HCI_CMD	18	Sent LE Connection Update
更新参数操作执行成功。controller发给host。这条命令之后，协议栈会发送命令给出去，给对面的BLE设备。
3963	15:53:01.566723	controller	host	HCI_EVT	13	Rcvd LE Meta (LE Connection Update Complete)
这边在等待BLE设备的回复，等待超时了。后面就断开。
5410	15:53:12.491452	controller	host	HCI_EVT	7	Rcvd Disconnect Complete
```

最后一个帧的内容，显示连接超时。

```
Frame 5410: 7 bytes on wire (56 bits), 7 bytes captured (56 bits)
    Encapsulation type: Bluetooth H4 with linux header (99)
    Arrival Time: Apr 25, 2019 23:53:12.491452000 China Standard Time
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1556207592.491452000 seconds
    [Time delta from previous captured frame: 0.001655000 seconds]
    [Time delta from previous displayed frame: 0.001655000 seconds]
    [Time since reference or first frame: 86.503400000 seconds]
    Frame Number: 5410
    Frame Length: 7 bytes (56 bits)
    Capture Length: 7 bytes (56 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    Point-to-Point Direction: Received (1)
    [Protocols in frame: bluetooth:hci_h4:bthci_evt]
Bluetooth
    [Source: controller]
    [Destination: host]
Bluetooth HCI H4
    [Direction: Rcvd (0x01)]
    HCI Packet Type: HCI Event (0x04)
Bluetooth HCI Event - Disconnect Complete
    Event Code: Disconnect Complete (0x05)
    Parameter Total Length: 4
    Status: Success (0x00)
    Connection Handle: 0x0002
    Reason: Connection Timeout (0x08)
```

而另外一个台设备， 却不会断开，第一个帧显示对面LE设备所能支持结构，其中Connection Parameters Request Procedure: False，表示不支持更新参数请求。

```
第一个帧：
1408	15:57:13.490071	controller	host	HCI_EVT	15	Rcvd LE Meta (LE Read Remote Used Features Complete)
第一个帧的内容：
Frame 1408: 15 bytes on wire (120 bits), 15 bytes captured (120 bits)
    Encapsulation type: Bluetooth H4 with linux header (99)
    Arrival Time: Apr 25, 2019 23:57:13.490071000 China Standard Time
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1556207833.490071000 seconds
    [Time delta from previous captured frame: 0.002823000 seconds]
    [Time delta from previous displayed frame: 0.002823000 seconds]
    [Time since reference or first frame: 39.218678000 seconds]
    Frame Number: 1408
    Frame Length: 15 bytes (120 bits)
    Capture Length: 15 bytes (120 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    Point-to-Point Direction: Received (1)
    [Protocols in frame: bluetooth:hci_h4:bthci_evt]
Bluetooth
    [Source: controller]
    [Destination: host]
Bluetooth HCI H4
    [Direction: Rcvd (0x01)]
    HCI Packet Type: HCI Event (0x04)
Bluetooth HCI Event - LE Meta
    Event Code: LE Meta (0x3e)
    Parameter Total Length: 12
    Sub Event: LE Read Remote Used Features Complete (0x04)
    Status: Success (0x00)
    Connection Handle: 0x0002
    Supported LE Features: 0x0000000000000001, LE Encryption
        .... .... .... .... .... .... .... .... .... .... .... .... .... .... .... ...1 = LE Encryption: True
        .... .... .... .... .... .... .... .... .... .... .... .... .... .... .... ..0. = Connection Parameters Request Procedure: False
        .... .... .... .... .... .... .... .... .... .... .... .... .... .... .... .0.. = Extended Reject Indication: False
        .... .... .... .... .... .... .... .... .... .... .... .... .... .... .... 0... = Slave-Initiated Features Exchange: False
        .... .... .... .... .... .... .... .... .... .... .... .... .... .... ...0 .... = Ping: False
        .... .... .... .... .... .... .... .... .... .... .... .... .... .... ..0. .... = Data Packet Length Extension: False
        .... .... .... .... .... .... .... .... .... .... .... .... .... .... .0.. .... = LL Privacy: False
        .... .... .... .... .... .... .... .... .... .... .... .... .... .... 0... .... = Extended Scanner Filter Policies: False
        .... .... .... .... .... .... .... .... .... .... .... .... .... ...0 .... .... = LE 2M PHY: False
        .... .... .... .... .... .... .... .... .... .... .... .... .... ..0. .... .... = Stable Modulation Index - Tx: False
        .... .... .... .... .... .... .... .... .... .... .... .... .... .0.. .... .... = Stable Modulation Index - Rx: False
        .... .... .... .... .... .... .... .... .... .... .... .... .... 0... .... .... = LE Coded PHY: False
        .... .... .... .... .... .... .... .... .... .... .... .... ...0 .... .... .... = LE Extended Advertising: False
        .... .... .... .... .... .... .... .... .... .... .... .... ..0. .... .... .... = LE Periodic Advertising: False
        .... .... .... .... .... .... .... .... .... .... .... .... .0.. .... .... .... = Channel Selection Algorithm #2: False
        .... .... .... .... .... .... .... .... .... .... .... .... 0... .... .... .... = Power Class 1: False
        .... .... .... .... .... .... .... .... .... .... .... ...0 .... .... .... .... = Minimum Number of Used Channels Procedure: False
        0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 000. .... .... .... .... = Reserved: 0x000000000000
    [Command in frame: 1393]
    [Pending in frame: 1398]
    [Pending-Response Delta: 54.527ms]
    [Command-Response Delta: 57.991ms]
```
虽然不支持更新参数请求，但是却发了更新参数请求过来，这个现象比较奇怪。
```
参数更新请求。
2698	15:57:19.440508	TexasIns_f0:d3:41 (Hon-RFID3)	localhost ()	L2CAP	21	Rcvd Connection Parameter Update Request
请求回复。
2702	15:57:19.441762	localhost ()	TexasIns_f0:d3:41 (Hon-RFID3)	L2CAP	15	Sent Connection Parameter Update Response (Accepted)
发送更新过去。
2705	15:57:19.442326	host	controller	HCI_CMD	18	Sent LE Connection Update
这条命令没有处理，应为没有controller通过hci发给host的命令回复，所以没有处理。就不会存在后面的等待超时。
```