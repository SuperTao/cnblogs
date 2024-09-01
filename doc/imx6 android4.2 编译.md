编译imx6dl android4.2的镜像，记录编译的命令。

#### Build Android Image

```
# Build Android images for i.MX6 SABRE-SD boards
cd ~/myandroid
source build/envsetup.sh
lunch sabresd_6dq-user
make
```

#### Build U-Boot Images

After you set up U-Boot using the steps outlined above, you can find the tool (mkimage) under tools/.

```
cd ~/myandroid/bootable/bootloader/uboot-imx
export ARCH=arm
export CROSS_COMPILE=~/myandroid/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin/arm-eabiCommand to build for i.MX 6Dual/6Quad SABRE-SD or i.MX 6DualLite SABRE-SD board is:
make distclean

# For i.MX 6Quad SABRE-SD:
make mx6q_sabresd_android_config
# For i.MX 6DualLite SABRE-SD:
make mx6dl_sabresd_android_config
make

# Command to build for i.MX 6 series SABRE-AI boards is:
make distclean
# For i.MX 6Quad SABRE-AI SD:
make mx6q_sabreauto_android_config
make
# For i.MX 6Quad SABRE-AI NAND:
make mx6q_sabreauto_nand_android_config
make
# For i.MX 6DualLite SABRE-AI SD:
make mx6dl_sabreauto_android_config
make
# For i.MX 6DualLite SABRE-AI NAND:
make mx6dl_sabreauto_nand_android_config
make
# For i.MX 6Solo SABRE-AI SD:
make mx6solo_sabreauto_android_config
make
# For i.MX 6Solo SABRE-AI NAND:
make mx6solo_sabreauto_nand_android_config
make
```

#### Build Kernel Image

```
export PATH=~/myandroid/bootable/bootloader/uboot-imx/tools:$PATH
cd ~/myandroid/kernel_imx
echo $ARCH && echo $CROSS_COMPILE
# Make sure you have those two environment variables set. If the two variables are not set, set them as:
export ARCH=arm
export CROSS_COMPILE=~/myandroid/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/bin/arm-eabi-
make imx6_android_defconfig
# Generate ".config" according to default config file under arch/arm/configs.
$ make uImage
```

#### Build boot.img

```
# Boot image for SABRE-SD board
cd ~/myandroid
source build/envsetup.sh
lunch sabresd_6dq-user
make bootimage
```

#### Build system.img

```
cd ~/myandroid
source build/envsetup.sh
lunch sabresd_6dq-user
make systemimage
```

Tony Liu

2017-2-23, Shenzhen