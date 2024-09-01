imx6开机启动就进入download模式，有的板子进入文件系统之后会进入download模式。查看datasheet，Chapter 8 System Boot查找原因,记录于此。

freescale论坛有关于这个问题的讨论，有硬件也有软件方面的原因。

#### 参考链接

　　https://community.nxp.com/thread/316232

　　https://community.nxp.com/thread/338433

#### boot mode pin settings

```
8.2.1 Boot mode pin settings
BOOT_MODE is initialized by sampling the BOOT_MODE0 and BOOT_MODE1
inputs on the rising edge of POR_B. After these inputs are sampled, their subsequent
state does not affect the contents of the BOOT_MODE internal register. The state of the
internal BOOT_MODE register may be read from the BMOD[1:0] field of the SRC Boot
Mode Register (SRC_SBMR2). The available boot modes are: Boot From Fuses, serial
boot via USB, and Internal Boot. See the table below for settings
根据BOOT_MODE[1:0]引脚的值选择启动类型
BOOT_MODE[1:0]      Boot Type
00                  Boot From Fuses
01                  Serial Downloader
10                  Internal Boot
11                  Reserved

在POR_B引脚的上升沿读取BOOT_MODE0和BOOT_MODE1的值作为启动的模式。

如果boot一开始就进入download模式，应该查看POR_B上升沿时，BOOT_MODE, BOOT_CFG引脚配置的值是否正确。
```

#### Boot devices(Internal Boot)

```
当选择为Internal Boot时，根据BOOT_CFG1[7:4]选择启动设备的类型，如果没有选择的设备，则进入download模式。 

NOR Flash with External Interface Module (EIM), located on CS0, 16-bit bus width
• OneNAND Flash with EIM interface, located on CS0, 16-bits bus width
• Raw NAND (MLC and SLC), and Toggle-mode NAND flash through GPMI-2
interface. Page sizes of 2 Kbyte, 4 Kbyte and 8 Kbyte. Bus widths of 8-bit with 2
through 40-bit BCH Hardware ECC (Error Correction) are supported.
• SD/MMC/eSD/SDXC/eMMC4.4 via USDHC interface, supporting high capacity
cards
• EEPROM boot via SPI (serial flash) and I2C(via ECSPI and I2C blocks respectively)
The selection of external boot device type is controlled by BOOT_CFG1[7:4] eFUSEs.
See the table below for more details

BOOT_CFG1[7:4]      Boot Device
0000                NOR/OneNAND (EIM)
0001                Reserved
0011                Serial ROM (I2C/SPI)
010x                SD/eSD/SDXC
011x                MMC/eMMC
1xxx                Raw NAND
```

#### BOOT_CFG其他引脚

```
BOOT_CFG1,BOOT_CFG2,BOOT_CFG3,BOOT_CFG4引脚进一步确定其他的参数。

例如sd/emmc,如下：
8.5.3 Expansion Device
The ROM supports booting from MMC/eMMC and SD/eSD compliant devices.
8.5.3.1 Expansion Device eFUSE Configuration
SD/MMC/eSD/eMMC/SDXC boot can be performed using either USDHC ports, based
on setting of the BOOT_CFG2[4:3] (Port Select) fuse or it's associated GPIO input value
at boot. All USDHC ports support eMMC4.3 and eMMC4.4 fast boot.

BOOT_CFG2[7:5]         选择SD/EMMC的参数
        SD/eSD/SDXC (BOOT_CFG1[5]=0)
        Bus Width
        xx0 - 1-bit
        xx1 - 4-bit
        SD Calibration Step
        00x - 1 delay cells
        01x - 1 delay cells
        10x - 2 delay cells
        11x - 3 delay cells
        MMC/eMMC (BOOT_CFG1[5]=1)
        000 - 1-bit
        001 - 4-bit
        010 - 8-bit
        101 - 4-bit DDR (MMC 4.4)
        110 - 8-bit DDR (MMC 4.4)
        Else - reserved.
BOOT_CFG2[4:3]    选择从哪个设备启动
        01 - USDHC-2
        10 - USDHC-3
        11 - USDHC-4

```
#### Author

Tony Liu

2016-10-31, Shenzhen