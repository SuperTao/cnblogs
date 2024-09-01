工作中需要将imx6的android系统从SD卡启动，所以就分析了MfgTool中的脚本，分析android的分区情况，并尝试自己操作，竟然成功了，记录于此。

### 参考文档

　　http://www.kancloud.cn/digest/imx6-android/148864

　　http://m.codes51.com/article/detail_239610_4.html
　　
### sd卡重新分区

　　分区使用MfgTool中的mksdcard-android.sh脚本。下面对其进行分析。
    
　　需要将SD卡umount才能够从新进行分区。

```
#!/bin/bash
# 运行 sh mksdcard-android.sh /dev/mmcblk0
# partition size in MB
BOOTLOAD_RESERVE=8  # bootloader
BOOT_ROM_SIZE=8     
SYSTEM_ROM_SIZE=512 # system.img
CACHE_SIZE=512      # cache
RECOVERY_ROM_SIZE=8 # recovery
VENDER_SIZE=8       # vendor
MISC_SIZE=8
#显示帮助信息
help() {

bn=`basename $0`
cat << EOF
usage $bn <option> device_node

options:
  -h                displays this help message
  -s                only get partition size
  -np               not partition.
  -f                flash android image.
EOF

}

# check the if root?
userid=`id -u`
if [ $userid -ne "0" ]; then
    echo "you're not root?"
    exit
fi

# parse command line
moreoptions=1
node="na"
cal_only=0
flash_images=0
not_partition=0
not_format_fs=0
while [ "$moreoptions" = 1 -a $# -gt 0 ]; do
    case $1 in
        -h) help; exit ;;
        -s) cal_only=1 ;;
        -f) flash_images=1 ;;
        -np) not_partition=1 ;;
        -nf) not_format_fs=1 ;;
        *)  moreoptions=0; node=$1 ;;
    esac
    [ "$moreoptions" = 0 ] && [ $# -gt 1 ] && help && exit
    [ "$moreoptions" = 1 ] && shift
done

if [ ! -e ${node} ]; then
    help
    exit
fi

# node /dev/mmcblk0

# call sfdisk to create partition table
# get total card size
seprate=40
# 查看分区大小,字节数
total_size=`sfdisk -s ${node}`
# 将字节转为M
total_size=`expr ${total_size} / 1024`
boot_rom_sizeb=`expr ${BOOT_ROM_SIZE} + ${BOOTLOAD_RESERVE}`    # 8M + 8M = 16M
#                    512M +  512M +  8M  + 8M + 40M = 1240M
extend_size=`expr ${SYSTEM_ROM_SIZE} + ${CACHE_SIZE} + ${VENDER_SIZE} + ${MISC_SIZE} + ${seprate}`
#  data_size  =      total_size -  16M  -  8M - 1240M +  40M = total_size - 1224M
data_size=`expr ${total_size} - ${boot_rom_sizeb} - ${RECOVERY_ROM_SIZE} - ${extend_size} + ${seprate}`
# 这一部分只是显示计算的大小
# create partitions
if [ "${cal_only}" -eq "1" ]; then
cat << EOF
BOOT   : ${boot_rom_sizeb}MB
RECOVERY: ${RECOVERY_ROM_SIZE}MB
SYSTEM : ${SYSTEM_ROM_SIZE}MB
CACHE  : ${CACHE_SIZE}MB
DATA   : ${data_size}MB
MISC   : ${MISC_SIZE}MB
EOF
exit
fi

# 删除分区表
# destroy the partition table
dd if=/dev/zero of=${node} bs=1024 count=1

# 创建分区
# sfdisk 命令 man 文档中有记录。
# Id is given in hex, without the 0x prefix, or is [E|S|L|X], 
# where L (LINUX_NATIVE (83)) is the default, S is LINUX_SWAP (82), 
# E is EXTENDED_PARTITION (5), and X is LINUX_EXTENDED (85).
# 指定分区大小，以及分区的类型，83，LINUX_NATIVE，5：扩展分区
# 查案每种文件系统类型，可以通过命令 sfdisk -T 查看 

sfdisk --force -uM ${node} << EOF
,${boot_rom_sizeb},83       # 16M
,${RECOVERY_ROM_SIZE},83    # 8M
,${extend_size},5           # 1240M
,${data_size},83            # total_size - 1224M
,${SYSTEM_ROM_SIZE},83      # 512M
,${CACHE_SIZE},83           # 512M
,${VENDER_SIZE},83          # 8M
,${MISC_SIZE},83            # 8M
EOF

# adjust the partition reserve for bootloader.
# if you don't put the uboot on same device, you can remove the BOOTLOADER_ERSERVE
# to have 8M space.
# the minimal sylinder for some card is 4M, maybe some was 8M
# just 8M for some big eMMC 's sylinder
# 将/dev/mmcblk0分区之后的第一个分区进行调整
sfdisk --force -uM ${node} -N1 << EOF
${BOOTLOAD_RESERVE},${BOOT_ROM_SIZE},83
EOF

# For MFGTool Notes:
# MFGTool use mksdcard-android.tar store this script
# if you want change it.
# do following:
#   tar xf mksdcard-android.sh.tar
#   vi mksdcard-android.sh 
#   [ edit want you want to change ]
#   rm mksdcard-android.sh.tar; tar cf mksdcard-android.sh.tar mksdcard-android.sh
```

### 镜像拷贝到SD卡 

　　这些内容也是从Mfgtool的ucl2.mxl文件中提取出来的。

　　制作成脚本repartition.sh，如下所示：

```
#!/bin/sh
# Tony Liu  2016-8-3

IMAGE_DIR="./image/"            #镜像文件存放的目录
UBOOT=$IMAGE_DIR/u-boot.bin
BOOT_IMG=$IMAGE_DIR/boot.img
SYSTEM_IMG=$IMAGE_DIR/system.img
RECOVERY_IMG=$IMAGE_DIR/recovery.img

DEVICE=/dev/sdb                 #SD卡在linux中的设备节点，视实际情况而定

MKSDCARD_SCRIPT=./mksdcard-android.sh   # android分区的脚本
# 将SD卡分区   
sh $MKSDCARD_SCRIPT $DEVICE
# 对设备写0，每次块大小是512字节，从2 block的位置开始写，写2000次
# 这里我猜想前面的1024字节应该是留给分区表的。
dd if=/dev/zero of=$DEVICE bs=512 seek=2 count=2000         #Clean U-Bootenvironment
# 写入u-boot.img。从u-boot.img开始2 block(skip=2)的位置开始读取数据
# 写入的地址是设备偏移2block(seek=2)的位置(1M),块大小512字节。
dd if=$UBOOT of=$DEVICE bs=512 seek=2 skip=2    #write U-Boot to sdcard
# 写入boot.img
dd if=$BOOT_IMG of=${DEVICE}1            #write boot.img

# 将SD卡的第4个分区格式化位ext4文件系统，并指定卷标名称为data
mkfs.ext4 -L data ${DEVICE}4             #Formatting sd partition

mkfs.ext4 -L system ${DEVICE}5           #Formatting system partition

mkfs.ext4 -L cache -O^extent ${DEVICE}6  #Formatting cache partition

mkfs.ext4 -L vender ${DEVICE}7           #Formatting data partition

mkfs.ext4 ${DEVICE}8                     #Formatting misc partition
# 写入system.img
dd if=$SYSTEM_IMG of=${DEVICE}5 bs=512   #Sending and writting system.img
# 写入recovery.img
dd if=$RECOVERY_IMG of=${DEVICE}2 bs=512 #Sending and writting recovery.img</CMD>
```

### 更改kernel启动位置

　　imx6从SD卡uboot启动之后，需要在SD卡中的uboot指定内核运行的SD卡序号，否者会运行开发板的emmc中。

　　可以在uboot中通过"mmc list"，查看有几块mmc。使用"booti mmc1",更改mmc序号，看是否能从SD卡启动，来判断SD卡的编号。

　　我的板子上SD卡的序号是mmc1。将uboot配置文件中mmc2更改为mmc1。这样一来，就选择从SD卡的内核启动。

　　vi mx6dl_sabresd_android.h

```
#define CONFIG_ANDROID_RECOVERY_BOOTCMD_MMC  \
"booti mmc1 recovery"
// Tony 2016-8-3
//"booti mmc2 recovery"
#define CONFIG_ANDROID_RECOVERY_CMD_FILE "/recovery/command"
#define CONFIG_INITRD_TAG

#undef CONFIG_LOADADDR
#undef CONFIG_RD_LOADADDR
#undef CONFIG_EXTRA_ENV_SETTINGS


#define CONFIG_LOADADDR     0x10800000  /* loadaddr env var */
#define CONFIG_RD_LOADADDR      0x11000000

#define CONFIG_INITRD_TAG
#define CONFIG_EXTRA_ENV_SETTINGS                   \
        "netdev=eth0\0"                     \
        "ethprime=FEC0\0"                   \
        "fastboot_dev=mmc1\0"                   \
        "bootcmd=booti mmc1\0"                  \
        "splashimage=0x30000000\0"              \
        "splashpos=m,m\0"                   \
        "android_test=keyvalue\0"                   \
        "bootargs=console=ttymxc0,115200 init=/init video=mxcfb0:dev=ldb,bpp=32 video=mxcfb1:off video=mxcfb2:off fbmem=40M fb0base=0x27b00000 vmalloc=400M androidboot.console=ttymxc0 androidboot.hardware=freescale\0" \
        "lvds_num=1\0"         
#endif
```
### 更改ramdisk挂载位置

　　内核更改之后，需要将文件系统挂载到SD卡的分区上。

　　根据自己SD卡编号进行更改/dev/block/mmcblk后面的序号，我的是1。

　　vi fstab.freescale

```
# Android fstab file.
# <src>             <mnt_point>  <type>  <mnt_flags>                                                                         <fs_mgr_flags>
# The filesystem that contains the filesystem checker binary (typically /system) cannot
# specify MF_CHECK, and must come before any filesystems that do specify MF_CHECK

#Tony add for SD card boot
/dev/block/mmcblk1p5    /system  ext4    ro                                                                               wait
/dev/block/mmcblk1p4    /data    ext4    nosuid,nodev,nodiratime,noatime,nomblk_io_submit,noauto_da_alloc,errors=panic    wait,encryptable=footer
/dev/block/mmcblk1p6    /cache   ext4    nosuid,nodev,nomblk_io_submit                                        wait
/dev/block/mmcblk1p7    /device  ext4    ro,nosuid,nodev                                                  wait
```

　　运行脚本输出内容：

```
Tony@Tony:~/imx6_sdcard_boot$ sudo ./repartition.sh 
###  首先进行SD卡分区
1+0 records in
1+0 records out
1024 bytes (1.0 kB) copied, 0.00902186 s, 114 kB/s
Checking that no-one is using this disk right now ...
OK

Disk /dev/sdb: 1022 cylinders, 245 heads, 62 sectors/track

sfdisk: ERROR: sector 0 does not have an msdos signature
 /dev/sdb: unrecognized partition table type
Old situation:
No partitions found
Warning: given size (6520) exceeds max allowable size (6460)
New situation:
Units = mebibytes of 1048576 bytes, blocks of 1024 bytes, counting from 0

   Device Boot Start   End    MiB    #blocks   Id  System
/dev/sdb1         0+    22-    23-     22784+  83  Linux
/dev/sdb2        22+    37-    15-     15190   83  Linux
/dev/sdb3        37+  1119-  1083-   1108870    5  Extended
/dev/sdb4      1119+  7639-  6520-   6676005   83  Linux
/dev/sdb5        37+   556-   520-    531649+  83  Linux
/dev/sdb6       556+  1075-   520-    531649+  83  Linux
/dev/sdb7      1075+  1090-    15-     15189+  83  Linux
/dev/sdb8      1090+  1105-    15-     15189+  83  Linux
Warning: partition 4 extends past end of disk
Successfully wrote the new partition table

Re-reading the partition table ...

If you created or changed a DOS partition, /dev/foo7, say, then use dd(1)
to zero the first 512 bytes:  dd if=/dev/zero of=/dev/foo7 bs=512 count=1
(See fdisk(8).)
Checking that no-one is using this disk right now ...
OK

Disk /dev/sdb: 1022 cylinders, 245 heads, 62 sectors/track
Old situation:
Units = mebibytes of 1048576 bytes, blocks of 1024 bytes, counting from 0

   Device Boot Start   End    MiB    #blocks   Id  System
/dev/sdb1         0+    22-    23-     22784+  83  Linux
/dev/sdb2        22+    37-    15-     15190   83  Linux
/dev/sdb3        37+  1119-  1083-   1108870    5  Extended
/dev/sdb4      1119+  7639-  6520-   6676005   83  Linux
/dev/sdb5        37+   556-   520-    531649+  83  Linux
/dev/sdb6       556+  1075-   520-    531649+  83  Linux
/dev/sdb7      1075+  1090-    15-     15189+  83  Linux
/dev/sdb8      1090+  1105-    15-     15189+  83  Linux
New situation:
Units = mebibytes of 1048576 bytes, blocks of 1024 bytes, counting from 0

   Device Boot Start   End    MiB    #blocks   Id  System
/dev/sdb1         7+    22-    15-     15190   83  Linux
/dev/sdb2        22+    37-    15-     15190   83  Linux
/dev/sdb3        37+  1119-  1083-   1108870    5  Extended
/dev/sdb4      1119+  7639-  6520-   6676005   83  Linux
/dev/sdb5        37+   556-   520-    531649+  83  Linux
/dev/sdb6       556+  1075-   520-    531649+  83  Linux
/dev/sdb7      1075+  1090-    15-     15189+  83  Linux
/dev/sdb8      1090+  1105-    15-     15189+  83  Linux
Warning: partition 4 extends past end of disk
Successfully wrote the new partition table

Re-reading the partition table ...

If you created or changed a DOS partition, /dev/foo7, say, then use dd(1)
to zero the first 512 bytes:  dd if=/dev/zero of=/dev/foo7 bs=512 count=1
(See fdisk(8).)
#-------------------------------------------------------------------------------
# 上面的内容是SD卡从新分区，下面进行dd文件拷贝
2000+0 records in
2000+0 records out
1024000 bytes (1.0 MB) copied, 2.21602 s, 462 kB/s   # dd命令还能够测读写速度
533+1 records in
533+1 records out
273172 bytes (273 kB) copied, 1.33912 s, 204 kB/s
9916+0 records in
9916+0 records out
5076992 bytes (5.1 MB) copied, 11.7538 s, 432 kB/s
mke2fs 1.42 (29-Nov-2011)
Filesystem label=data
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
413712 inodes, 1654536 blocks
82726 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=1694498816
51 block groups
32768 blocks per group, 32768 fragments per group
8112 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done 

mke2fs 1.42 (29-Nov-2011)
Filesystem label=system
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
33280 inodes, 132912 blocks
6645 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=138412032
5 block groups
32768 blocks per group, 32768 fragments per group
6656 inodes per group
Superblock backups stored on blocks: 
	32768, 98304

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.42 (29-Nov-2011)
Filesystem label=cache
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
33280 inodes, 132912 blocks
6645 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=138412032
5 block groups
32768 blocks per group, 32768 fragments per group
6656 inodes per group
Superblock backups stored on blocks: 
	32768, 98304

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.42 (29-Nov-2011)
Filesystem label=vender
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
3808 inodes, 15188 blocks
759 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=15728640
2 block groups
8192 blocks per group, 8192 fragments per group
1904 inodes per group
Superblock backups stored on blocks: 
	8193

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.42 (29-Nov-2011)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
3808 inodes, 15188 blocks
759 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=15728640
2 block groups
8192 blocks per group, 8192 fragments per group
1904 inodes per group
Superblock backups stored on blocks: 
	8193

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

573440+0 records in
573440+0 records out
293601280 bytes (294 MB) copied, 61.2452 s, 4.8 MB/s
10580+0 records in
10580+0 records out
5416960 bytes (5.4 MB) copied, 14.8084 s, 366 kB/s
```

### Author

Tony Liu

2016-8-3, Shenzhen