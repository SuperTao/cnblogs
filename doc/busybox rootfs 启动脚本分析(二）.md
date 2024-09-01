上次分析了busybox的启动脚本，这次分析一下init.d中一些脚本的内容。

#### 参考链接

http://www.cnblogs.com/helloworldtoyou/p/6169678.html

http://m.blog.chinaunix.net/uid-20678569-id-1574823.html

https://www.ibm.com/developerworks/cn/linux/l-cn-udev/

http://blog.csdn.net/future_fighter/article/details/3862795

#### mount-proc-sys

```
#!/bin/sh

# Copyright 2006-2007 Freescale Semiconductor, Inc.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.

if [ "$1" = "start" -o "$1" = "restart" ]
then
    echo "Mounting /proc and /sys 
	# -t指定文件系统类型， -n不把安装记录在/etc/mtab文件中"
    mount -n -t proc proc /proc  	# 挂载 proc
    mount -n -t sysfs sys /sys		# 挂载 sysfs
fi
```

#### udev

```
#!/bin/sh
PATH=$PATH:/sbin:/bin

if [ ! -x /sbin/udevd ]
then
    exit 0
fi

case "$1" in		# start
    start)
        echo "" > /proc/sys/kernel/hotplug

        mount -n -o mode=0755 -t tmpfs tmpfs /dev	# 挂载tmpfs到/dev

        # Create static device nodes in /dev
        mknod /dev/console c 5 1	# 创建/dev/console节点
        mknod /dev/null c 1 3		# 创建/dev/null节点

        echo "Starting the hotplug events dispatcher udevd"
        udevd --daemon				# 启动udev,监听内核的uevent,自动创建或删除/dev中的文件

        echo "Synthesizing initial hotplug events"
        udevtrigger					# 扫描sysfs文件系统，生成相应的硬件设备hotplug事件
        udevsettle --timeout=300	# 查看udev事件队列，等队列内事件全部处理完毕才退出。

        mkdir /dev/pts
        mount -n -t devpts devpts /dev/pts		# 挂载 devpts

        mkdir /dev/shm
        ;;
    stop)		# stop
        ;;
    reload)
        udevcontrol --reload_rules
        ;;
    *)
        echo "Usage: /etc/rc.d/init.d/udev {start|stop|reload}"
        echo
        exit 1
        ;;
esac

exit 0
```

#### hostname

```
#!/bin/sh

if [ "$1" = "start" ]
then
    if [ -x /bin/hostname -o -x /usr/bin/hostname ]
    then
        echo Setting the hostname to $HOSTNAME
        hostname $HOSTNAME				# 设置主机名
    fi
fi
```

#### depmod

```
#!/bin/sh

if [ ! -x /sbin/depmod ]
then
    exit 0
fi

if [ "$1" = "start"  -a ! -s /lib/modules/`uname -r`/modules.dep ]	# 判断modules.dep内容是否为空
then
    echo Running depmod
    /sbin/depmod -a		# 使用depmod创建modules.dep文件
						# depmod读取/lib/modules/`uname -r`中的所有模块一来关系，并记录到modules.dep文件中。
fi
```

#### modules

```
#!/bin/sh

if [ "$1" = "start" -a -x /sbin/modprobe -a "$MODLIST" ]	
then
    for i in $MODLIST
    do
        echo Loading module $i
        modprobe $i				# 根据MODLIST中的列表安装模块
    done
fi
```

#### filesystems

```
#!/bin/sh

if [ "$1" = "stop" ]
then
    echo Unmounting filesystems
    umount -a -r		# -a: All  of the file systems described in /etc/mtab are unmounted.
						# -r: In case unmounting fails, try to remount read-only.
    mount -o remount -r %root% /		# 重新挂载分区
    [ -x /sbin/swapoff ] && swapoff -a	# 关闭配置文件/etc/fstab中所有系统交换分区
fi

if [ "$1" = "start" ]
then
    echo Mounting filesystems
    if [ "$TMPFS" = "tmpfs" ]
    then
        mount -n -t $TMPFS shm /dev/shm
    fi
    if [ -n "$TMPFS" ]
    then
        mount -n -t $TMPFS rwfs /mnt/rwfs -o size=$TMPFS_SIZE
    fi
    if [ "$READONLY_FS" != "y" ]
    then
        mount -n -o remount -w %root% /
        NFSBOOT="`cat /proc/cmdline | grep -q /dev/nfs ; echo $?`"
        if [ "$NFSBOOT" == "0"  -a  -n "$RAMDIRS" ]
        then
            echo "Booted NFS, not relocating: $RAMDIRS"
            RAMDIRS=""
        fi
    else
        # initramfs, ramdisks, others? come up read/write by default
        mount -n -o remount -r %root% /
        RAMDIRS="$RAMDIRS /tmp /etc /var"
    fi
    if [ -n "$RAMDIRS" ]
    then
        for i in $RAMDIRS
        do
            if [ ! -e /mnt/rwfs/$i ]
            then
                cp -a $i /mnt/rwfs/
                mount -n -o bind /mnt/rwfs/$i $i
            fi
        done
    fi
    if [ -e /etc/mtab ]
    then
        rm -f /etc/mtab
    fi
    ln -s /proc/mounts /etc/mtab	# 创建/proc/mounts软连接 /etc/mtab
    if [ ! -d /dev/pts ]
    then
        mkdir /dev/pts		# 创建/dev/pts
    fi
    mount -a
fi
```

#### inetd

```
#!/bin/sh

if [ ! -x /usr/sbin/inetd ]
then
    exit 0
fi

if [ "$1" = "stop" -o "$1" = "restart" ]
then                                                                            
    echo "Stopping inetd: "
    killall inetd
fi

if [ "$1" = "start" -o "$1" = "restart" ]
then
    echo "Starting inetd: "
    /usr/sbin/inetd $INETD_ARGS
fi
```

Tony Liu

2016-12-18， Shenzhen