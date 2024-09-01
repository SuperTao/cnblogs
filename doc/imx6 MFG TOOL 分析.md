之前分析过mfgtool的内容，最近从官网下载，返现新版的mfgtool工具将imx6各种版本的linux/android都使用一个工具进行烧录。所以从新分析一下。

新版与旧版的一个区别是烧写使用的uboot后缀是.imx，而不是原来的.bin。以后缀.imx结尾的uboot在镜像开头1k的地方添加了IVT表。

本文分析的MFG_TOOL版本是：IMX6_L5.1_2.1.0_MFG_TOOL。

#### 参考链接

---

http://www.cnblogs.com/helloworldtoyou/p/5730750.html

#### vbscript

---

首先双击对应的vbscript启动mfgtool工具。一般从vbs的名称就能够看出是烧录何种cpu以及存储设备的类型。例如：

mfgtool2-android-mx6q-sabresd-emmc.vbs

表示烧录的镜像运行的系统是android, cpu是mx6q, 存储设备类型为emmc.

详细分析如下：

```
Set wshShell = CreateObject("WScript.shell")
wshShell.run "mfgtool2.exe -c ""linux"" -l ""eMMC-Android"" -s ""board=sabresd""  -s ""folder=sabresd"" -s ""soc=6q"" -s ""mmc=3"" -s ""data_type="""
Set wshShell = Nothing

参数的含义：
-c: 芯片配置文件夹名称 linux
-l: list名称，对应ucl2.xml文件中的LIST标签的name，后面会说明。
-s: 变量名称, cfg.ini以及ucl2.xml中会用到。
```

#### cfg.ini

---

脚本运行, mfgtool工具启动，解读cfg.ini文件，上面的vbs脚本中的参数会覆盖cfg.ini文件中的参数。
```
[profiles]
chip = Linux

[platform]
board = SabreSD

# vbs中 -l ""eMMC-Android"" 已经制定为eMMC-Android，所以下面的LIST无效。
[LIST]
name = SDCard

[variable]
board = sabresd
mmc = 0
sxuboot=17x17arm2
sxdtb=17x17-arm2
7duboot=sabresd
7ddtb=sdb
6uluboot=14x14ddr3arm2
6uldtb=14x14-ddr3-arm2
ldo=
plus=
initramfs=fsl-image-mfgtool-initramfs-imx_mfgtools.cpio.gz.u-boot
seek = 1
sxnor=qspi2
7dnor=qspi1
6ulnor=qspi1
nor_part=0
```

#### ucl2.xml

---

根据 vbs和cfg.ini文件中的参数指定的LIST名称，选择正确的操作进行进行烧录。

这个目录里面有很多文件,烧录的时候要用到的uboot,内核，文件系统到RAM中，运行

* 首先烧录Profiles/Linux/OS Firmware/firmware文件夹的镜像到RAM中运行系统。

* 系统运行之后烧录Profiles/Linux/OS Firmware/files/android/sabresd/中的镜像到emmc中


```
<!-- 首先判断不同芯片USB的vid和pid-->
<CFG>
    <STATE name="BootStrap" dev="MX6SL" vid="15A2" pid="0063"/>
    <STATE name="BootStrap" dev="MX6D" vid="15A2" pid="0061"/>
    <STATE name="BootStrap" dev="MX6Q" vid="15A2" pid="0054"/>
    <STATE name="BootStrap" dev="MX6SX" vid="15A2" pid="0071"/>
    <STATE name="BootStrap" dev="MX6UL" vid="15A2" pid="007D"/>
    <STATE name="BootStrap" dev="MX7D" vid="15A2" pid="0076"/>
    <STATE name="Updater"   dev="MSC" vid="066F" pid="37FF"/>
  </CFG>

<LIST name="eMMC-Android" desc="Choose eMMC as media">

<!-- 将两个%号之间的参数使用vbs脚本和cfg.ini文件中的值进行替换 -->  

	<!-- file ="firmware/u-boot-imx6qsabresd_sd.imx" -- >
	<CMD state="BootStrap" type="boot" body="BootStrap" file ="firmware/u-boot-imx6q%plus%%board%_sd.imx" ifdev="MX6Q">Loading U-boot</CMD>
	<CMD state="BootStrap" type="boot" body="BootStrap" file ="firmware/u-boot-imx6dl%board%_sd.imx" ifdev="MX6D">Loading U-boot</CMD>
	<CMD state="BootStrap" type="boot" body="BootStrap" file ="firmware/u-boot-imx6sx%board%_emmc.imx" ifdev="MX6SX">Loading U-boot</CMD>
	<CMD state="BootStrap" type="boot" body="BootStrap" file ="firmware/u-boot-imx7d%7duboot%_sd.imx" ifdev="MX7D">Loading U-boot</CMD>
	<CMD state="BootStrap" type="boot" body="BootStrap" file ="firmware/u-boot-imx6ul%6uluboot%_emmc.imx" ifdev="MX6UL">Loading U-boot</CMD>
		
<!-- 下载镜像到CPU的RAM中直接运行，当系统跑起来之后再烧录镜像到EMMC中。不同的CPU型号下载的地址不同 -->
	<CMD state="BootStrap" type="load" file="firmware/zImage" address="0x12000000"
		loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" ifdev="MX6Q MX6D">Loading Kernel.</CMD>
	<CMD state="BootStrap" type="load" file="firmware/zImage" address="0x80800000"
		loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" ifdev="MX6SL MX6SX MX7D MX6UL">Loading Kernel.</CMD>

	<CMD state="BootStrap" type="load" file="firmware/%initramfs%" address="0x12C00000"
		loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" ifdev="MX6Q MX6D">Loading Initramfs.</CMD>
	<CMD state="BootStrap" type="load" file="firmware/%initramfs%" address="0x83800000"
		loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" ifdev="MX6SL MX6SX MX7D MX6UL">Loading Initramfs.</CMD>

	<CMD state="BootStrap" type="load" file="firmware/zImage-imx6q%plus%-%board%%ldo%.dtb" address="0x18000000"
		loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" ifdev="MX6Q">Loading device tree.</CMD>
	<CMD state="BootStrap" type="load" file="firmware/zImage-imx6dl-%board%%ldo%.dtb" address="0x18000000"
		loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" ifdev="MX6D">Loading device tree.</CMD>

	<CMD state="BootStrap" type="jump" > Jumping to OS image. </CMD>
    
<!-- 烧录镜像到EMMC中 --> 
  	<!-- create partition -->
<!-- 先对EMMC进行分区 -->
	<CMD state="Updater" type="push" body="send" file="mksdcard-android%data_type%.sh.tar">Sending partition shell</CMD>
	<CMD state="Updater" type="push" body="$ tar xf $FILE "> Partitioning...</CMD>
    <!-- data_type = "" , mmc = '3' -->
	<CMD state="Updater" type="push" body="$ sh mksdcard-android%data_type%.sh /dev/mmcblk%mmc%"> Partitioning...</CMD>
	
	<!-- burn uboot -->
	<CMD state="Updater" type="push" body="$ dd if=/dev/zero of=/dev/mmcblk%mmc% bs=1k seek=384 conv=fsync count=129">clear u-boot arg</CMD>
	<CMD state="Updater" type="push" body="$ echo 0 > /sys/block/mmcblk%mmc%boot0/force_ro">access boot partition 1</CMD>
	
	<CMD state="Updater" type="push" body="send" file="files/android/%folder%/u-boot-imx%soc%%plus%.imx" >Sending u-boot.bin</CMD>
	
<!-- 将uboot烧录到emmc的boot0分区,在系统盘符里分区的序号是从0开始，也就是boot0, boot1 -->

	<CMD state="Updater" type="push" body="$ dd if=$FILE of=/dev/mmcblk%mmc%boot0 bs=512 seek=2">write U-Boot to sd card</CMD>
	<CMD state="Updater" type="push" body="$ echo 1 > /sys/block/mmcblk%mmc%boot0/force_ro"> re-enable read-only access </CMD>
<!-- 将boot1分区使能，那么启动的时候就会从boot1分区启动,emmc寄存器序号是从1开始,boot1,boot2.这里的boot1就是指上面的boot0 -->
	<CMD state="Updater" type="push" body="$ mmc bootpart enable 1 1 /dev/mmcblk%mmc%">enable boot partion 1 to boot</CMD>

	<CMD state="Updater" type="push" body="$ ls -l /dev/mmc* ">Formatting sd partition</CMD>

	<CMD state="Updater" type="push" body="send" file="files/android/%folder%/boot-imx%soc%%plus%%ldo%.img" >Sending and writting boot.img</CMD>
	
	<CMD state="Updater" type="push" body="$ dd if=$FILE of=/dev/mmcblk%mmc%p1">write boot.img</CMD>


	<CMD state="Updater" type="push" body="$ mkfs.ext4  -E nodiscard /dev/mmcblk%mmc%p5">Formatting system partition</CMD>
	<CMD state="Updater" type="push" body="$ mkfs.ext4  -E nodiscard /dev/mmcblk%mmc%p6">Formatting cache partition</CMD>

	<CMD state="Updater" type="push" body="$ mkfs.ext4  -E nodiscard /dev/mmcblk%mmc%p7">Formatting device partition</CMD>
	<CMD state="Updater" type="push" body="$ mount -o remount,size=512M rootfs /">change size of tmpfs</CMD>
	<CMD state="Updater" type="push" body="send" file="files/android/%folder%/system.img" >Sending system.img</CMD>
	<CMD state="Updater" type="push" body="$ simg2img $FILE /dev/mmcblk%mmc%p5">writting sparse system.img</CMD>
	
	<!-- Write userdata.img is optional, for some customer this is needed, but it's optional. -->
	<!-- Also, userdata.img will have android unit test, you can use this to do some auto test. -->
	<!--	<CMD state="Updater" type="push" onError="ignore" body="pipe dd of=/dev/mmcblk0p7" file="file/android/userdate.img"> Sending userdata.img(optional) </CMD>
	<CMD state="Updater" type="push" body="frf">flush the memory.</CMD>  -->
	<CMD state="Updater" type="push" body="pipe dd of=/dev/mmcblk%mmc%p2 bs=512" file="files/android/%folder%/recovery-imx%soc%%plus%%ldo%.img">Sending and writting recovery.img</CMD>
	
	<CMD state="Updater" type="push" body="$ sync">Sync file system</CMD>
	<CMD state="Updater" type="push" body="frf">Finishing rootfs write</CMD>

	<CMD state="Updater" type="push" body="$ echo Update Complete!">Done</CMD>
	
</LIST>
```

Tony Liu

2016-11-11, Shenzhen