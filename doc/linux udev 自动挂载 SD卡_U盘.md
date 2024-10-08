本文记录使用udev自动挂载SD卡和U盘的方法。

#### 参考链接

http://blog.chinaunix.net/uid-26119896-id-5211736.html

#### 添加udev规则

创建文件/etc/udev/rules.d/11-add-usb.rules

```
# SD卡自动挂载
ACTION=="add",GOTO="farsight", KERNEL=="mmcblk[0-9]p[0-9]", RUN+="/etc/mount-sd.sh %k", LABEL="farsight"

# U盘自动挂载
ACTION=="add",GOTO="farsight",KERNEL=="sd[a-z][0-9]",RUN+="/etc/mount-usb.sh %k",LABEL="farsight"
```

/etc/udev/rules.d/11-add-remove.rules

```
# 卸载SD卡
ACTION=="remove",GOTO="farsight", SUBSYSTEM=="block",GOTO="farsight", KERNEL=="mmcblk[0-9]p[0-9]",RUN+="/etc/umount-sd.sh", LABEL="farsight"

# 卸载U盘
ACTION=="remove",GOTO="farsight",SUBSYSTEM=="block",GOTO="farsight",KERNEL=="sd[a-z][0-9]",RUN+="/etc/umount-usb.sh",LABEL="farsight"
```

#### 创建挂载的目录

`mkdir /mnt/sd -p`

`mkdir /mnt/usb -p`

#### 添加脚本

创建脚本/etc/mount-sd.sh

```
#!/bin/sh
/bin/mount -t vfat /dev/$1 /mnt/sd
sync
```

添加可执行权限`chmod +x /etc/mount-sd.sh`

/etc/umount-sd.sh

```
#!/bin/sh
sync
umount /mnt/sd
```

`chmod +x /etc/umount-sd.sh`


/etc/mount-usb.sh

```
#!/bin/sh
mount  -t vfat /dev/$1 /mnt/usb
sync
```

`chmod +x /etc/mount-usb.sh`

/etc/umount-usb.sh

```
#!/bin/sh
sync
umount /mnt/usb
```

`chmod +x /etc/umount-usb.sh`

Tony Liu

2017-1-5, Shenzhen