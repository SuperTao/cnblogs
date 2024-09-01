imx6文件系统启动脚本分析。开机运行/sbin/init，读取/etc/inittab文件，进行初始化。

#### 参考链接 

http://blog.163.com/wghbeyond@126/blog/static/35166181201051483723579/

#### /etc/inittab
```
# see busybox-1.00rc2/examples/inittab for more examples
::sysinit:/etc/rc.d/rcS			# 系统启动的时候运行/etc/rc.d/rcS脚本
#::respawn:/etc/rc.d/rc_mxc.S
ttymxc0::once:/bin/login root		# debug口使用ttymxc0，之运行一次/bin/login，root是传入的参数。
				        # 这就是为什么登录之后退出就不能再登录。如果想再次运行/bin/login,将once改为respawn
#::once:/sbin/getty 115200 ttymxc0	# 也可以采用这种形式设置串口，不过这样就需要输入账户密码

::sysinit:/etc/rc.d/rc_gpu.S		# 系统启动的时候运行rc_gpu.S
::ctrlaltdel:/sbin/reboot 			# ctrl + alt + del 组合键是运行/sbin/reboot程序。
::shutdown:/etc/rc.d/rcS stop 		# 关机的时候运行 /etc/rc.d/rcS stop
::restart:/sbin/init				# 重启时运行/sbin/init
``` 
#### /etc/rc.d/rcS
```
#!/bin/sh

# minimal startup script, will work with msh (this is best available in
# MMUless format).

# load the configuration information
. /etc/rc.d/rc.conf			# 运行/etc/rc.d/rc.conf 脚本
mode=${1:-start}			# 当$1的值不存在或者为空，那么mode=start，否者mode=$1
if [ $mode = "start" ]
then
    services=$cfg_services
else
    services=$cfg_services_r
fi
cfg_services=${2:-$services}

# run the configured sequence
for i in $cfg_services
do
    if [ -x /etc/rc.d/init.d/$i ]	# 判断/etc/rc.d/init.d/中是否存在$i表示的可执行文件
    then                                                                        
        /etc/rc.d/init.d/$i $mode	# 运行/etc/rc.d/init.d中的shell脚本，start or stop 
    fi                                                                          
done

if [ $# -ge 2 ]	 			# 如果传入脚本的参数个数大于 2
then 
    exit 0				# 退出, 返回值为 0
fi
# show all kernel log messages
#echo 8 >  /proc/sys/kernel/printk

# run rc.local if present and executable
if [ -x /etc/rc.d/rc.local ]		# 判断rc.local是否存在，并且可执行
then 
    /etc/rc.d/rc.local $mode		# 运行rc.local, mode= start/stop
fi
```

#### /etc/rc.d/rc.conf

```
all_services="mount-proc-sys mdev udev hostname devfsd depmod modules filesystems syslog network inetd portmap dropbear sshd boa smb dhcpd settime fslgnome watchdog bluetooth gtk2 pango"
all_services_r="pango gtk2 bluetooth watchdog fslgnome settime dhcpd smb boa sshd dropbear portmap inetd network syslog filesystems modules depmod devfsd hostname udev mdev mount-proc-sys"
# 启动，运行顺序
cfg_services="mount-proc-sys  udev hostname  depmod modules filesystems   inetd            "
# 停止，停止顺序与启动顺序相反
cfg_services_r="            inetd   filesystems modules depmod  hostname udev  mount-proc-sys"

export HOSTNAME="freescale"
export NTP_SERVER=""
export MODLIST=""
export RAMDIRS=""
export TMPFS="tmpfs"
export TMPFS_SIZE="512k"
export READONLY_FS=""
export INETD_ARGS=""
export BOA_ARGS=""
export SMBD_ARGS=""
export NMBD_ARGS=""
export DHCP_ARG=""
export DEPLOYMENT_STYLE="RAMDISK"
export SYSCFG_DHCPC_CMD="udhcpc -b -i "
export DROPBEAR_ARGS=""
```

#### /etc/rc.d/rc.local

```
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here
if [ -x "/usr/bin/rpm" -a -e "/tmp/ltib" ]		# 是否存在可执行文件/usr/bin/rpm，以及/tmp/ltib
then
    echo "rebuilding rpm database"
    rm -rf /tmp/ltib					# 重新创建rpm数据库
    rpm --rebuilddb
fi

# fix up permissions
if [ -d /home/user ]					# 是否存在/home/user目录
then
    chown -R user.user /home/user 		        # 更改所有者,所在组
fi

# Add nodes when running under the hypervisor and static devices
if [ -r /sys/class/misc/fsl-hv/dev -a ! -r /dev/fsl-hv ]	 # 查看/sys/class/misc/fsl-hv/dev是否可读，并且/dev/fsl-hv不可读
then
   echo "creating hypervisor nodes"
   DEVID=`cat /sys/class/misc/fsl-hv/dev`
   if [ -n "$DEVID" ]						# 判断是否为空
   then
       MAJOR="${DEVID%:*}"
       MINOR="${DEVID##*:}"

       if [ \( "$MAJOR" -gt 0 \) -a \( "$MINOR" -gt 0 \) ]
       then
	   rm -f /dev/fsl-hv
	   mknod /dev/fsl-hv c $MAJOR $MINOR
       fi
   fi
   for i in 0 1 2 3 4 5 6 7
   do
       mknod /dev/hvc$i c 229 $i
   done
fi

# add the fm device nodes
if [ -n "$(cat /proc/devices | grep fm | sed 's/\([0-9]*\).*/\1/')" -a ! -r /dev/fm0 ]
then
    echo "creating fman device nodes"
    cd /usr/share/doc/fmd-uspace-01.01/test/
    sh fm_dev_create
    cd -
fi

for i in 0 1 2; do
    if [ -e /sys/class/graphics/fb$i ]; then
        chmod 0666 /sys/class/graphics/fb$i/pan
    fi
done
```

#### /etc/rc.d/rc_gpu.S

```
#!/bin/bash
CPUREV=$(cat /proc/cpuinfo | grep Revision | awk '{print $3}' | awk '{print substr($0,1,2)}')	# 查看cpu的版本，mx6dl读取的是61
FILEVG=/usr/lib/libOpenVG.so
FILEVG3D=/usr/lib/libOpenVG_3D.so
FILEVG355=/usr/lib/libOpenVG_355.so
echo 4 > /sys/module/galcore/parameters/gpu3DMinClock
if  [ -e $FILEVG3D ] && [ -e $FILEVG355 ]
then
  if  [ $CPUREV == "61" ] || [ $CPUREV == "63" ] || [ $CPUREV == "60" ] && [ -e  $FILEVG ]
  then
        rm -f $FILEVG
  fi  
  if [ $CPUREV == "61" ]
  then
        ln -s $FILEVG3D $FILEVG	  # 创建软连接
  fi  
  if [ $CPUREV == "63" ]
  then
        ln -s $FILEVG355 $FILEVG
  fi  
  if [ $CPUREV == "60" ]
  then
        ln -s $FILEVG355 $FILEVG
  fi  
fi
```

#### /etc/profile

登录的时候，输入完账户密码之后，调用/etc/profile

---

Tony Liu

2016-12-13, Shenzhen