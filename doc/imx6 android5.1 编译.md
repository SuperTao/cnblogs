imx6 android5.1 编译

记录一下编译imx6dl android的命令。

#### Android build

```
cd ~/myandroid
source build/envsetup.sh
lunch sabresd_6dq-user
make 2>&1 | tee build-log.txt
```

#### Building U-Boot images

```
cd ~/myandroid/bootable/bootloader/uboot-imx
export ARCH=arm
export CROSS_COMPILE=~/myandroid/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin/arm-eabi-
make distclean
#For i.MX 6Quad SABRE-SD:
make mx6qsabresdandroid_config
make
```

#### Building a kernel image

Kernel image is automatically built when building the Android root file system.

The following are the default Android build commands to build the kernel image:

```
export PATH=~/myandroid/bootable/bootloader/uboot-imx/tools:$PATH
cd ~/myandroid/kernel_imx
echo $ARCH && echo $CROSS_COMPILE
# Make sure that you have those two environment variables set. If the two variables are not set, set them as follows:
export ARCH=arm
export CROSS_COMPILE=~/myandroid/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin/arm-eabi-
# Generate ".config" according to default config file under arch/arm/configs.
# To build the kernel image for i.MX 6Dual/Quad, 6QuadPlus, 6DualLite, 6Solo, 6SoloLite, 6SoloX, and 7Dual
make imx_v7_android_defconfig
# To build the kernel image for i.MX 6Dual/Quad, 6QuadPlus, 6DualLite, and 6Solo
make uImage LOADADDR=0x10008000
# To build the kernel image for i.MX 6SoloLite
make uImage LOADADDR=0x80008000
# To build the kernel image for i.MX 6SoloX
make uImage LOADADDR=0x80008000
# To build the kernel uImage for i.MX 7Dual
make uImage LOADADDR=0x80008000
```

The kernel images are found in the folders: ~/myandroid/kernel_imx/arch/arm/boot/zImage and ~/myandroid/kernel_imx/arch/arm/boot/uImage.


#### Building boot.img

```
# Boot image for the i.MX 6Dual/6Quad SABRE-SD board
cd ~/myandroid
source build/envsetup.sh
lunch sabresd_6dq-user		# imx6dl
make bootimage
# Boot image for the i.MX 6Dual/6Quad/6QuadPlus SABRE-AI board
source build/envsetup.sh
lunch sabreauto_6q-user
make bootimage
# Boot image for the i.MX 6SoloLite EVK board
source build/envsetup.sh
lunch evk_6sl-user
make bootimage
# Boot image for the i.MX 6SoloX SABRE-SD board
source build/envsetup.sh
lunch sabresd_6sx-user
make bootimage
# Boot image for the i.MX 6SoloX SABRE-AI board
source build/envsetup.sh
lunch sabreauto_6sx-user
make bootimage
# Boot image for i.MX 7Dual SABRE-SD board
source build/envsetup.sh
lunch sabresd_7d-user
make bootimage
```

#### Building system.img

```
cd ~/myandroid
source build/envsetup.sh
lunch sabresd_6dq-user
make systemimage
```

Tony Liu

2017-2-21, Shenzhen