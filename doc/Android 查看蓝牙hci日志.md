最近在调试android连接ble设备，需要查看hci日志。记录一下方法。

```
1. 开发者选项->启用蓝牙HCI信息收集日志。

2. android 8版本，默认位置/data/misc/bluetooth/logs

/data/misc/bluetooth/logs # ls -l
total 3904
-rw-rw-r-- 1 bluetooth bluetooth      16 2019-04-25 17:27 btsnoop_hci.log
-rw-rw-r-- 1 bluetooth bluetooth      16 2019-04-25 17:27 btsnoop_hci.log.last
-rw-rw-r-- 1 bluetooth bluetooth 2755648 2019-04-25 18:16 hci_snoop20190425172740.cfa
-rw-rw-r-- 1 bluetooth bluetooth 1221959 2019-04-25 18:21 hci_snoop20190425173638.cfa

3. android 7， 默认位置 /sdcard/

4. 配置hci路径：
cat /etc/bluetooth/bt_statck.conf
# BtSnoop log output file
BtSnoopFileName=/sdcard/btsnoop_hci.log

5. 用wireshark等工具打开即可。
```