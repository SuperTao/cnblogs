使用otg端口进行usb slave的测试,需要安装g_file_storage.ko或者g_mass_storage.ko模块。

#### 参考链接

http://blog.csdn.net/freeman1975/article/details/46364319

http://linux-sunxi.org/USB_Gadget/Mass_storage

http://xlon.de/wiki/index.php?title=USB_Mass_Storage_Gadget_(or_MSG)

http://www.linux-usb.org/gadget/file_storage.html

--- 

必须编译成模块安装，并指定一块空间作为存储设备。

例如：

先分配一个1G的空间

dd if=/dev/zero of=/mass_storage bs=1M seek=1024 count=0

安装时，作为参数传给模块

modprobe g_mass_storage file=/mass_storage

安装成功后，插上usb线，PC机就会提示识别USB存储设备，需要进行格式化。

---

Tony Liu

2016-12-15， ShenZhen