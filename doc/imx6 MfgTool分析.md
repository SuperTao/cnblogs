解析freescale的MfgTool中的脚本,了解imx6, android系统的分区情况。

### 配置文件

　　1. cfg.ini

```
[profiles]
chip = MX6DL Linux Update
# 指定平台名称
[platform]
board = SabreSD
# 根据name名称指定ucl2.xml中运行的命令 
[LIST]
name = Android-SPI_NOR-EMMC
```
　　2. UICfg.ini

　　用于配置同时烧写的主板个数,MfgTool的说明文档中指出，支持同时烧写多个主板。 

```
[UICfg]
PortMgrDlg=1  

```
### emmc分区

　　android分区脚本mksdcard-android.sh

```
#!/bin/bash
# 运行 sh mksdcard-android.sh /dev/mmcblk0
# partition size in MB
BOOTLOAD_RESERVE=8	# bootloader
BOOT_ROM_SIZE=8		
SYSTEM_ROM_SIZE=512	# system.img
CACHE_SIZE=512		# cache
RECOVERY_ROM_SIZE=8 # recovery
VENDER_SIZE=8		# vendor
MISC_SIZE=8
#显示帮助信息
help() {

bn=`basename $0`
cat << EOF
usage $bn <option> device_node

options:
  -h				displays this help message
  -s				only get partition size
  -np 				not partition.
  -f 				flash android image.
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
boot_rom_sizeb=`expr ${BOOT_ROM_SIZE} + ${BOOTLOAD_RESERVE}`	# 16M
extend_size=`expr ${SYSTEM_ROM_SIZE} + ${CACHE_SIZE} + ${VENDER_SIZE} + ${MISC_SIZE} + ${seprate}`
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
,${boot_rom_sizeb},83		# boot_rom_sizeb
,${RECOVERY_ROM_SIZE},83
,${extend_size},5
,${data_size},83
,${SYSTEM_ROM_SIZE},83
,${CACHE_SIZE},83
,${VENDER_SIZE},83
,${MISC_SIZE},83
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

### 烧写镜像

　　烧写的过程就记录在ucl2.xml中，根据cfg.ini中指定的LIST的name值选择运行的命令，将系统镜像烧写到emmc的分区中。 

```
<!--
* Copyright (C) 2010-2013, Freescale Semiconductor, Inc. All Rights Reserved.
* The CFG element contains a list of recognized usb devices.
*  DEV elements provide a name, class, vid and pid for each device.
*
* Each LIST element contains a list of update instructions.
*  "Install" - Erase media and install firmware.
*  "Update" - Update firmware only.
*
* Each CMD element contains one update instruction of attribute type.
*  "pull" - Does UtpRead(body, file) transaction.
*  "push" - Does UtpWrite(body, file) transaction.
*  "drop" - Does UtpCommand(body) then waits for device to disconnect.
*  "boot" - Finds configured device, forces it to "body" device and downloads "file".
*  "find" - Waits for "timeout" seconds for the "body" device to connect.
*  "show" - Parse and show device info in "file".  
-->


<UCL>
  <CFG>
	<STATE name="BootStrap" dev="MX6D" vid="15A2" pid="0061"/>
	<STATE name="Updater"   dev="MSC" vid="066F" pid="37FF"/> 
  </CFG>
<!--
  The following Lists are for i.MX6Solo/DualLite chips
-->

<LIST name="MYZR-I.MX6-SPI_NOR & eMMC" desc="Choose SPI-NOR/eMMC as media">
    ......
</LIST> 

<LIST name="MYZR-I.MX6-SPI_NOR & SD card" desc="Choose SPI-NOR/SD as media">
    ...... 
</LIST>

<LIST name="MYZR-I.MX6-UBUNTU-SPI_NOR & eMMC" desc="Choose eMMC as media">
    ......
</LIST>


<LIST name="Android-SPI_NOR-eMMC" desc="Choose SPI-NOR and SD Rootfs as media"> 
    <!-- 按照官方文档说明更改的uboot，用于MfgTool烧写程序的时候使用 --> 
	<CMD state="BootStrap" type="boot" body="BootStrap" file ="myzr_u-boot.bin" >Loading U-boot</CMD>
	<CMD state="BootStrap" type="load" file="uImage" address="0x10800000"
        loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" >Loading Kernel.</CMD>
	<CMD state="BootStrap" type="load" file="initramfs.cpio.gz.uboot" address="0x10C00000"
        loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" >Loading Initramfs.</CMD>
	<CMD state="BootStrap" type="jump" > Jumping to OS image. </CMD> 
	
	<!--
	Please use "cat /proc/mtd" to check the right partitions for NAND
	,mtd0 and mtd1 are for SPI-NOR; mtd2 - mtd6 are for NAND
	-->
<!--	<CMD state="Updater" type="push" body="mknod class/mtd,mtd0,/dev/mtd0"/>
	<CMD state="Updater" type="push" body="mknod block,mtdblock0,/dev/mtdblock0,block"/> -->
<!----
	<CMD state="Updater" type="push" body="$ flash_erase /dev/mtd0 0 0">Erasing Boot partition</CMD>
	<CMD state="Updater" type="push" body="send" file="files/android/u-boot.bin">Sending U-Boot</CMD>
	<!-- 将u-boot.bin写到/dev/mtd0，每次读写的大小是512-->
	<CMD state="Updater" type="push" body="$ dd if=$FILE of=/dev/mtd0 bs=512">write U-Boot to SPI-NOR</CMD>
--->
	<CMD state="Updater" type="push" body="send" file="mksdcard-android.sh.tar">Sending partition shell</CMD>
	<CMD state="Updater" type="push" body="$ tar xf $FILE "> Partitioning...</CMD>
	<CMD state="Updater" type="push" body="$ sh mksdcard-android.sh /dev/mmcblk0"> Partitioning...</CMD>
	<!-- 使用脚本将/dev/mmcblk0进行分区-->
	<CMD state="Updater" type="push" body="$ ls -l /dev/mmc* ">Formatting sd partition</CMD>
	<!-- burn the uboot: -->
	<CMD state="Updater" type="push" body="send" file="files/android/u-boot.bin">Sending U-Boot</CMD>
	<!-- 将/dev/mmcblk0清0，每次操作512字节，操作次数200次，操作位置，从偏移2block的地方开始输出-->
	<CMD state="Updater" type="push" body="$ dd if=/dev/zero of=/dev/mmcblk0 bs=512 seek=2 count=2000">Clean U-Bootenvironment</CMD>
	<!-- 将u-boot.bin写道/dev/mmcblk0，每次写512字节，从uboot偏移地址是2 block的地方开始读取，从/dev/mmcblk0偏移2block的地方开始输出 -->
	<CMD state="Updater" type="push" body="$ dd if=$FILE of=/dev/mmcblk0 bs=512 seek=2 skip=2">write U-Boot to sdcard</CMD>
	<!-- burn the uImage: -->
	<CMD state="Updater" type="push" body="send" file="files/android/boot.img">Sending kernel uImage</CMD>
	<!-- 将boot.img写入到/dev/mmcblk0p1 -->
	<CMD state="Updater" type="push" body="$ dd if=$FILE of=/dev/mmcblk0p1">write boot.img</CMD>
	<CMD state="Updater" type="push" body="frf">flush the memory.</CMD>
	<!-- 将/dev/mmcblk0p4格式化位ext4文件系统，并指定卷标名称为data-->
	<CMD state="Updater" type="push" body="$ mkfs.ext4 -L data /dev/mmcblk0p4">Formatting sd partition</CMD>
	<!-- 将/dev/mmcblk0p5格式化位ext4文件系统，并指定卷标名称为system-->
	<CMD state="Updater" type="push" body="$ mkfs.ext4 -L system /dev/mmcblk0p5">Formatting system partition</CMD>
	<!-- 将/dev/mmcblk0p6格式化位ext4文件系统，并指定卷标名称为cache-->
	<CMD state="Updater" type="push" body="$ mkfs.ext4 -L cache -O^extent /dev/mmcblk0p6">Formatting cache partition</CMD>
	<!-- 将/dev/mmcblk0p7格式化位ext4文件系统，并指定卷标名称为vender-->
	<CMD state="Updater" type="push" body="$ mkfs.ext4 -L vender /dev/mmcblk0p7">Formatting data partition</CMD>
	<CMD state="Updater" type="push" body="frf">flush the memory.</CMD>
	<CMD state="Updater" type="push" body="$ mkfs.ext4 /dev/mmcblk0p8">Formatting misc partition</CMD>
	<!-- 将system.img些到/dev/mmcblk0p5中,由于文件比较大，所以是通过读取管道再写入-->
	<CMD state="Updater" type="push" body="pipe dd of=/dev/mmcblk0p5 bs=512" file="files/android/system.img">Sending and writting system.img</CMD>
	<CMD state="Updater" type="push" body="frf">flush the memory.</CMD> 
	<!-- Write userdata.img is optional, for some customer this is needed, but it's optional. -->
	<!-- Also, userdata.img will have android unit test, you can use this to do some auto test. -->
<!--	<CMD state="Updater" type="push" onError="ignore" body="pipe dd of=/dev/mmcblk0p7" file="file/android/userdate.img"> Sending userdata.img(optional) </CMD>
	<CMD state="Updater" type="push" body="frf">flush the memory.</CMD> -->
	<CMD state="Updater" type="push" body="pipe dd of=/dev/mmcblk0p2 bs=512" file="files/android/recovery.img">Sending and writting recovery.img</CMD>
	<CMD state="Updater" type="push" body="frf">Finishing rootfs write</CMD>
	<CMD state="Updater" type="push" body="$ echo Update Complete!">Done</CMD> 
</LIST> 
</UCL>
```

### Author

Tony Liu

2016-8-2, Shenzhen