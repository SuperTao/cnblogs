imx6开启启动之后，运行板子上的ROM程序。ROM确定启动的设备，进行一些初始化，然后读取IVT，进行寄存器初始化,最后运行uboot/cpu/arm_cortexa8/start.S中的_start函数。

参考

http://blog.csdn.net/njuitjf/article/details/20563867

http://blog.csdn.net/sz_zh/article/details/7930341

截取IMX6SDLRM.pdf部分内容
```
8.6.1 Image Vector Table and Boot Data
# ROM读取IVT中内容。
The Image Vector Table (IVT) is the data structure that the ROM reads from the boot
device supplying the program image containing the required data components to perform
a successful boot.
# IVT包含DCD的入口点和其他的入口点，给ROM使用。
# IVT在设备中的地址是固定的，这样ROM才能够找到。
The IVT includes the program image entry point, a pointer to Device Configuration Data
(DCD) and other pointers used by the ROM during the boot process.The ROM locates
the IVT at a fixed address that is determined by the boot device connected to the Chip.
The IVT offset from the base address and initial load region size for each boot device
type is defined in the table below. The location of the IVT is the only fixed requirement
by the ROM. The remainder or the image memory map is flexible and is determined by
the contents of the IVT.
# IVT表在不同设备中的偏移地址和空间的大小
Table 8-24. Image Vector Table Offset and Initial Load Region Size
Boot Device Type 		Image Vector Table Offset 		Initial Load Region Size
NOR 					4 Kbyte = 0x1000 bytes 			Entire Image Size
NAND 					1 Kbyte = 0x400 bytes 			4 Kbyte
OneNAND 				256 bytes = 0x100 bytes 		1 Kbyte
SD/MMC/eSD/eMMC/SDXC 	1 Kbyte = 0x400 bytes 			4 Kbyte
I2C/SPI EEPROM 			1 Kbyte = 0x400 bytes 			4 Kbyte

# DCD
8.6.2 Device Configuration Data (DCD)
# 开启启动的时候根据需要更改寄存器的默认值,方便一些外设一开机就配置
Upon reset, the Chip uses the default register values for all peripherals in the system.
However, these settings typically are not ideal for achieving optimal system performance
and there are even some peripherals that must be configured before they can be used.

The DCD is configuration information contained in a Program Image, external to the
ROM, that the ROM interprets to configure various peripherals on the Chip.
```

flash_header.S分析

```
#include <config.h>
#include <asm/arch/mx6.h>

#ifdef	CONFIG_FLASH_HEADER
#ifndef CONFIG_FLASH_HEADER_OFFSET
# error "Must define the offset of flash header"
#endif

#define CPU_2_BE_32(l) \
       ((((l) & 0x000000FF) << 24) | \
	(((l) & 0x0000FF00) << 8)  | \
	(((l) & 0x00FF0000) >> 8)  | \
	(((l) & 0xFF000000) >> 24))

#define MXC_DCD_ITEM(i, addr, val)   \
dcd_node_##i:                        \
        .word CPU_2_BE_32(addr) ;     \		# 存放寄存器地址, 4字节
        .word CPU_2_BE_32(val)  ;     \		# 存放设置的值,	  4字节

.section ".text.flasheader", "x"
	b	_start
	.org	CONFIG_FLASH_HEADER_OFFSET

# ivt header
ivt_header:       .word 0x402000D1 /* Tag=0xD1, Len=0x0020, Ver=0x40 */
# entry
app_code_jump_v:  .word _start
reserv1:          .word 0x0
dcd_ptr:          .word dcd_hdr
boot_data_ptr:	  .word boot_data
self_ptr:         .word ivt_header
#ifdef CONFIG_SECURE_BOOT
app_code_csf:     .word __hab_data
#else
app_code_csf:     .word 0x0
#endif
reserv2:          .word 0x0

boot_data:        .word TEXT_BASE
#ifdef CONFIG_SECURE_BOOT
image_len:        .word __hab_data_end - TEXT_BASE + CONFIG_FLASH_HEADER_OFFSET
#else
image_len:        .word _end_of_copy  - TEXT_BASE + CONFIG_FLASH_HEADER_OFFSET
#endif
plugin:           .word 0x0

#if defined CONFIG_MX6DL_DDR3
#if defined CONFIG_DDR_32BIT
dcd_hdr:          .word 0x40E001D2 /* Tag=0xD2, Len=59*8 + 4 + 4, Ver=0x40 */
write_dcd_cmd:    .word 0x04DC01CC /* Tag=0xCC, Len=59*8 + 4, Param=0x04 */
# 各种DDR寄存器的初始化
MXC_DCD_ITEM(1, IOMUXC_BASE_ADDR + 0x774, 0x000C0000)        // DDR3
MXC_DCD_ITEM(2, IOMUXC_BASE_ADDR + 0x754, 0x00000000)        // DDR Pull/Kepper Disable

MXC_DCD_ITEM(3, IOMUXC_BASE_ADDR + 0x4ac, 0x00000030)
MXC_DCD_ITEM(4, IOMUXC_BASE_ADDR + 0x4b0, 0x00000030)

MXC_DCD_ITEM(5, IOMUXC_BASE_ADDR + 0x464, 0x00000030)
MXC_DCD_ITEM(6, IOMUXC_BASE_ADDR + 0x490, 0x00000030)
MXC_DCD_ITEM(7, IOMUXC_BASE_ADDR + 0x74c, 0x00000030)

MXC_DCD_ITEM(8, IOMUXC_BASE_ADDR + 0x494, 0x00000030)

MXC_DCD_ITEM(9, IOMUXC_BASE_ADDR + 0x4a0, 0x00000000)

MXC_DCD_ITEM(10, IOMUXC_BASE_ADDR + 0x4b4, 0x00000030)
MXC_DCD_ITEM(11, IOMUXC_BASE_ADDR + 0x4b8, 0x00000030)
MXC_DCD_ITEM(12, IOMUXC_BASE_ADDR + 0x76c, 0x00000030)

MXC_DCD_ITEM(13, IOMUXC_BASE_ADDR + 0x750, 0x00020000)

MXC_DCD_ITEM(14, IOMUXC_BASE_ADDR + 0x4bc, 0x00000030)
MXC_DCD_ITEM(15, IOMUXC_BASE_ADDR + 0x4c0, 0x00000030)
MXC_DCD_ITEM(16, IOMUXC_BASE_ADDR + 0x4c4, 0x00000030)
MXC_DCD_ITEM(17, IOMUXC_BASE_ADDR + 0x4c8, 0x00000030)

MXC_DCD_ITEM(18, IOMUXC_BASE_ADDR + 0x760, 0x00020000)

MXC_DCD_ITEM(19, IOMUXC_BASE_ADDR + 0x764, 0x00000030)
MXC_DCD_ITEM(20, IOMUXC_BASE_ADDR + 0x770, 0x00000030)
MXC_DCD_ITEM(21, IOMUXC_BASE_ADDR + 0x778, 0x00000030)
MXC_DCD_ITEM(22, IOMUXC_BASE_ADDR + 0x77c, 0x00000030)

MXC_DCD_ITEM(23, IOMUXC_BASE_ADDR + 0x470, 0x00000030)
MXC_DCD_ITEM(24, IOMUXC_BASE_ADDR + 0x474, 0x00000030)
MXC_DCD_ITEM(25, IOMUXC_BASE_ADDR + 0x478, 0x00000030)
MXC_DCD_ITEM(26, IOMUXC_BASE_ADDR + 0x47c, 0x00000030)

MXC_DCD_ITEM(27, MMDC_P0_BASE_ADDR + 0x800, 0xA1390003)

MXC_DCD_ITEM(28, MMDC_P0_BASE_ADDR + 0x80c, 0x001F001F)
MXC_DCD_ITEM(29, MMDC_P0_BASE_ADDR + 0x810, 0x001F001F)

MXC_DCD_ITEM(30, MMDC_P0_BASE_ADDR + 0x83c, 0x42190219)
MXC_DCD_ITEM(31, MMDC_P0_BASE_ADDR + 0x840, 0x017B0177)
MXC_DCD_ITEM(32, MMDC_P0_BASE_ADDR + 0x848, 0x4B4D4E4D)
MXC_DCD_ITEM(33, MMDC_P0_BASE_ADDR + 0x850, 0x3F3E2D36)

MXC_DCD_ITEM(34, MMDC_P0_BASE_ADDR + 0x81c, 0x33333333)
MXC_DCD_ITEM(35, MMDC_P0_BASE_ADDR + 0x820, 0x33333333)
MXC_DCD_ITEM(36, MMDC_P0_BASE_ADDR + 0x824, 0x33333333)
MXC_DCD_ITEM(37, MMDC_P0_BASE_ADDR + 0x828, 0x33333333)

MXC_DCD_ITEM(38, MMDC_P0_BASE_ADDR + 0x8b8, 0x00000800)

MXC_DCD_ITEM(39, MMDC_P0_BASE_ADDR + 0x004, 0x0002002D)
MXC_DCD_ITEM(40, MMDC_P0_BASE_ADDR + 0x008, 0x00333030)
MXC_DCD_ITEM(41, MMDC_P0_BASE_ADDR + 0x00c, 0x3F435313)
MXC_DCD_ITEM(42, MMDC_P0_BASE_ADDR + 0x010, 0xB66E8B63)
MXC_DCD_ITEM(43, MMDC_P0_BASE_ADDR + 0x014, 0x01FF00DB)
MXC_DCD_ITEM(44, MMDC_P0_BASE_ADDR + 0x018, 0x00001740)

MXC_DCD_ITEM(45, MMDC_P0_BASE_ADDR + 0x01c, 0x00008000)
MXC_DCD_ITEM(46, MMDC_P0_BASE_ADDR + 0x02c, 0x000026d2)

MXC_DCD_ITEM(47, MMDC_P0_BASE_ADDR + 0x030, 0x00431023)
MXC_DCD_ITEM(48, MMDC_P0_BASE_ADDR + 0x040, 0x00000017)

MXC_DCD_ITEM(49, MMDC_P0_BASE_ADDR + 0x000, 0x83190000)

MXC_DCD_ITEM(50, MMDC_P0_BASE_ADDR + 0x01c, 0x04008032)
MXC_DCD_ITEM(51, MMDC_P0_BASE_ADDR + 0x01c, 0x00008033)
MXC_DCD_ITEM(52, MMDC_P0_BASE_ADDR + 0x01c, 0x00048031)
MXC_DCD_ITEM(53, MMDC_P0_BASE_ADDR + 0x01c, 0x05208030)
MXC_DCD_ITEM(54, MMDC_P0_BASE_ADDR + 0x01c, 0x04008040)

MXC_DCD_ITEM(55, MMDC_P0_BASE_ADDR + 0x020, 0x00005800)
MXC_DCD_ITEM(56, MMDC_P0_BASE_ADDR + 0x818, 0x00011117)

MXC_DCD_ITEM(57, MMDC_P0_BASE_ADDR + 0x004, 0x0002556d)
MXC_DCD_ITEM(58, MMDC_P0_BASE_ADDR + 0x404, 0x00011006)

MXC_DCD_ITEM(59, MMDC_P0_BASE_ADDR + 0x01c, 0x00000000)
#else /* i.MX6DL 64BIT-DDR */
dcd_hdr:          .word 0x40A002D2 /* Tag=0xD2, Len=83*8 + 4 + 4, Ver=0x40 */
write_dcd_cmd:    .word 0x049C02CC /* Tag=0xCC, Len=83*8 + 4, Param=0x04 */

# IOMUXC_BASE_ADDR  = 0x20e0000
# DDR IO TYPE
MXC_DCD_ITEM(1, IOMUXC_BASE_ADDR + 0x774, 0x000c0000)
MXC_DCD_ITEM(2, IOMUXC_BASE_ADDR + 0x754, 0x00000000)
# Clock
MXC_DCD_ITEM(3, IOMUXC_BASE_ADDR + 0x4ac, 0x00000030)
MXC_DCD_ITEM(4, IOMUXC_BASE_ADDR + 0x4b0, 0x00000030)
# Address
MXC_DCD_ITEM(5, IOMUXC_BASE_ADDR + 0x464, 0x00000030)
MXC_DCD_ITEM(6, IOMUXC_BASE_ADDR + 0x490, 0x00000030)
MXC_DCD_ITEM(7, IOMUXC_BASE_ADDR + 0x74c, 0x00000030)
# Control
MXC_DCD_ITEM(8, IOMUXC_BASE_ADDR + 0x494, 0x00000030)

MXC_DCD_ITEM(9, IOMUXC_BASE_ADDR + 0x4a0, 0x00000000)

MXC_DCD_ITEM(10, IOMUXC_BASE_ADDR + 0x4b4, 0x00000030)
MXC_DCD_ITEM(11, IOMUXC_BASE_ADDR + 0x4b8, 0x00000030)
MXC_DCD_ITEM(12, IOMUXC_BASE_ADDR + 0x76c, 0x00000030)
# Data Strobe
MXC_DCD_ITEM(13, IOMUXC_BASE_ADDR + 0x750, 0x00020000)

MXC_DCD_ITEM(14, IOMUXC_BASE_ADDR + 0x4bc, 0x00000030)
MXC_DCD_ITEM(15, IOMUXC_BASE_ADDR + 0x4c0, 0x00000030)
MXC_DCD_ITEM(16, IOMUXC_BASE_ADDR + 0x4c4, 0x00000030)
MXC_DCD_ITEM(17, IOMUXC_BASE_ADDR + 0x4c8, 0x00000030)
MXC_DCD_ITEM(18, IOMUXC_BASE_ADDR + 0x4cc, 0x00000030)
MXC_DCD_ITEM(19, IOMUXC_BASE_ADDR + 0x4d0, 0x00000030)
MXC_DCD_ITEM(20, IOMUXC_BASE_ADDR + 0x4d4, 0x00000030)
MXC_DCD_ITEM(21, IOMUXC_BASE_ADDR + 0x4d8, 0x00000030)

MXC_DCD_ITEM(22, IOMUXC_BASE_ADDR + 0x760, 0x00020000)

MXC_DCD_ITEM(23, IOMUXC_BASE_ADDR + 0x764, 0x00000030)
MXC_DCD_ITEM(24, IOMUXC_BASE_ADDR + 0x770, 0x00000030)
MXC_DCD_ITEM(25, IOMUXC_BASE_ADDR + 0x778, 0x00000030)
MXC_DCD_ITEM(26, IOMUXC_BASE_ADDR + 0x77c, 0x00000030)
MXC_DCD_ITEM(27, IOMUXC_BASE_ADDR + 0x780, 0x00000030)
MXC_DCD_ITEM(28, IOMUXC_BASE_ADDR + 0x784, 0x00000030)
MXC_DCD_ITEM(29, IOMUXC_BASE_ADDR + 0x78c, 0x00000030)
MXC_DCD_ITEM(30, IOMUXC_BASE_ADDR + 0x748, 0x00000030)

MXC_DCD_ITEM(31, IOMUXC_BASE_ADDR + 0x470, 0x00000030)
MXC_DCD_ITEM(32, IOMUXC_BASE_ADDR + 0x474, 0x00000030)
MXC_DCD_ITEM(33, IOMUXC_BASE_ADDR + 0x478, 0x00000030)
MXC_DCD_ITEM(34, IOMUXC_BASE_ADDR + 0x47c, 0x00000030)
MXC_DCD_ITEM(35, IOMUXC_BASE_ADDR + 0x480, 0x00000030)
MXC_DCD_ITEM(36, IOMUXC_BASE_ADDR + 0x484, 0x00000030)
MXC_DCD_ITEM(37, IOMUXC_BASE_ADDR + 0x488, 0x00000030)
MXC_DCD_ITEM(38, IOMUXC_BASE_ADDR + 0x48c, 0x00000030)

# MMDC_P0_BASE_ADDR = 0x021b0000
# MMDC_P1_BASE_ADDR = 0x021b4000
# Calibrations
# ZQ
MXC_DCD_ITEM(39, MMDC_P0_BASE_ADDR + 0x800, 0xa1390003)

# write leveling
MXC_DCD_ITEM(40, MMDC_P0_BASE_ADDR + 0x80c, 0x001F001F)
MXC_DCD_ITEM(41, MMDC_P0_BASE_ADDR + 0x810, 0x001F001F)
MXC_DCD_ITEM(42, MMDC_P1_BASE_ADDR + 0x80c, 0x001F001F)
MXC_DCD_ITEM(43, MMDC_P1_BASE_ADDR + 0x810, 0x001F001F)
# DQS gating, read delay, write delay calibration values
# based on calibration compare of 0x00ffff00
MXC_DCD_ITEM(44, MMDC_P0_BASE_ADDR + 0x83c, 0x42480248)
MXC_DCD_ITEM(45, MMDC_P0_BASE_ADDR + 0x840, 0x0211020B)
MXC_DCD_ITEM(46, MMDC_P1_BASE_ADDR + 0x83C, 0x417F0211)
MXC_DCD_ITEM(47, MMDC_P1_BASE_ADDR + 0x840, 0x015D0166)

MXC_DCD_ITEM(48, MMDC_P0_BASE_ADDR + 0x848, 0x4B4C504D)
MXC_DCD_ITEM(49, MMDC_P1_BASE_ADDR + 0x848, 0x494C4F48)

MXC_DCD_ITEM(50, MMDC_P0_BASE_ADDR + 0x850, 0x3F3F2E31)
MXC_DCD_ITEM(51, MMDC_P1_BASE_ADDR + 0x850, 0x2B35382B)

MXC_DCD_ITEM(52, MMDC_P0_BASE_ADDR + 0x81c, 0x33333333)
MXC_DCD_ITEM(53, MMDC_P0_BASE_ADDR + 0x820, 0x33333333)
MXC_DCD_ITEM(54, MMDC_P0_BASE_ADDR + 0x824, 0x33333333)
MXC_DCD_ITEM(55, MMDC_P0_BASE_ADDR + 0x828, 0x33333333)
MXC_DCD_ITEM(56, MMDC_P1_BASE_ADDR + 0x81c, 0x33333333)
MXC_DCD_ITEM(57, MMDC_P1_BASE_ADDR + 0x820, 0x33333333)
MXC_DCD_ITEM(58, MMDC_P1_BASE_ADDR + 0x824, 0x33333333)
MXC_DCD_ITEM(59, MMDC_P1_BASE_ADDR + 0x828, 0x33333333)

MXC_DCD_ITEM(60, MMDC_P0_BASE_ADDR + 0x8b8, 0x00000800)
MXC_DCD_ITEM(61, MMDC_P1_BASE_ADDR + 0x8b8, 0x00000800)
# MMDC init:
# in DDR3, 64-bit mode, only MMDC0 is initiated:
MXC_DCD_ITEM(62, MMDC_P0_BASE_ADDR + 0x004, 0x0002002D)
MXC_DCD_ITEM(63, MMDC_P0_BASE_ADDR + 0x008, 0x00333030)
MXC_DCD_ITEM(64, MMDC_P0_BASE_ADDR + 0x00c, 0x3F435313)
MXC_DCD_ITEM(65, MMDC_P0_BASE_ADDR + 0x010, 0xB66E8B63)
MXC_DCD_ITEM(66, MMDC_P0_BASE_ADDR + 0x014, 0x01FF00DB)
MXC_DCD_ITEM(67, MMDC_P0_BASE_ADDR + 0x018, 0x00081740)

MXC_DCD_ITEM(68, MMDC_P0_BASE_ADDR + 0x01c, 0x00008000)
MXC_DCD_ITEM(69, MMDC_P0_BASE_ADDR + 0x02c, 0x000026d2)
MXC_DCD_ITEM(70, MMDC_P0_BASE_ADDR + 0x030, 0x00431023)
MXC_DCD_ITEM(71, MMDC_P0_BASE_ADDR + 0x040, 0x00000027)

MXC_DCD_ITEM(72, MMDC_P0_BASE_ADDR + 0x000, 0x831A0000)

# Initialize 2GB DDR3 - Micron MT41J128M
MXC_DCD_ITEM(73, MMDC_P0_BASE_ADDR + 0x01c, 0x04008032)
MXC_DCD_ITEM(74, MMDC_P0_BASE_ADDR + 0x01c, 0x00008033)
MXC_DCD_ITEM(75, MMDC_P0_BASE_ADDR + 0x01c, 0x00048031)
MXC_DCD_ITEM(76, MMDC_P0_BASE_ADDR + 0x01c, 0x05208030)
MXC_DCD_ITEM(77, MMDC_P0_BASE_ADDR + 0x01c, 0x04008040)

MXC_DCD_ITEM(78, MMDC_P0_BASE_ADDR + 0x020, 0x00005800)

MXC_DCD_ITEM(79, MMDC_P0_BASE_ADDR + 0x818, 0x00011117)
MXC_DCD_ITEM(80, MMDC_P1_BASE_ADDR + 0x818, 0x00011117)

MXC_DCD_ITEM(81, MMDC_P0_BASE_ADDR + 0x004, 0x0002556d)
MXC_DCD_ITEM(82, MMDC_P1_BASE_ADDR + 0x404, 0x00011006)
MXC_DCD_ITEM(83, MMDC_P0_BASE_ADDR + 0x01c, 0x00000000)
#endif
#else  /* i.MX6Q */
dcd_hdr:          .word 0x40a002D2 /* Tag=0xD2, Len=83*8 + 4 + 4, Ver=0x40 */
write_dcd_cmd:    .word 0x049c02CC /* Tag=0xCC, Len=83*8 + 4, Param=0x04 */

/* DCD */
MXC_DCD_ITEM(1, IOMUXC_BASE_ADDR + 0x798, 0x000C0000)
MXC_DCD_ITEM(2, IOMUXC_BASE_ADDR + 0x758, 0x00000000)

MXC_DCD_ITEM(3, IOMUXC_BASE_ADDR + 0x588, 0x00000030)
MXC_DCD_ITEM(4, IOMUXC_BASE_ADDR + 0x594, 0x00000030)

MXC_DCD_ITEM(5, IOMUXC_BASE_ADDR + 0x56c, 0x00000030)
MXC_DCD_ITEM(6, IOMUXC_BASE_ADDR + 0x578, 0x00000030)
MXC_DCD_ITEM(7, IOMUXC_BASE_ADDR + 0x74c, 0x00000030)

MXC_DCD_ITEM(8, IOMUXC_BASE_ADDR + 0x57c, 0x00000030)

MXC_DCD_ITEM(9, IOMUXC_BASE_ADDR + 0x58c, 0x00000000)
MXC_DCD_ITEM(10, IOMUXC_BASE_ADDR + 0x59c, 0x00000030)
MXC_DCD_ITEM(11, IOMUXC_BASE_ADDR + 0x5a0, 0x00000030)
MXC_DCD_ITEM(12, IOMUXC_BASE_ADDR + 0x78c, 0x00000030)

MXC_DCD_ITEM(13, IOMUXC_BASE_ADDR + 0x750, 0x00020000)

MXC_DCD_ITEM(14, IOMUXC_BASE_ADDR + 0x5a8, 0x00000030)
MXC_DCD_ITEM(15, IOMUXC_BASE_ADDR + 0x5b0, 0x00000030)
MXC_DCD_ITEM(16, IOMUXC_BASE_ADDR + 0x524, 0x00000030)
MXC_DCD_ITEM(17, IOMUXC_BASE_ADDR + 0x51c, 0x00000030)
MXC_DCD_ITEM(18, IOMUXC_BASE_ADDR + 0x518, 0x00000030)
MXC_DCD_ITEM(19, IOMUXC_BASE_ADDR + 0x50c, 0x00000030)
MXC_DCD_ITEM(20, IOMUXC_BASE_ADDR + 0x5b8, 0x00000030)
MXC_DCD_ITEM(21, IOMUXC_BASE_ADDR + 0x5c0, 0x00000030)

MXC_DCD_ITEM(22, IOMUXC_BASE_ADDR + 0x774, 0x00020000)

MXC_DCD_ITEM(23, IOMUXC_BASE_ADDR + 0x784, 0x00000030)
MXC_DCD_ITEM(24, IOMUXC_BASE_ADDR + 0x788, 0x00000030)
MXC_DCD_ITEM(25, IOMUXC_BASE_ADDR + 0x794, 0x00000030)
MXC_DCD_ITEM(26, IOMUXC_BASE_ADDR + 0x79c, 0x00000030)
MXC_DCD_ITEM(27, IOMUXC_BASE_ADDR + 0x7a0, 0x00000030)
MXC_DCD_ITEM(28, IOMUXC_BASE_ADDR + 0x7a4, 0x00000030)
MXC_DCD_ITEM(29, IOMUXC_BASE_ADDR + 0x7a8, 0x00000030)
MXC_DCD_ITEM(30, IOMUXC_BASE_ADDR + 0x748, 0x00000030)

MXC_DCD_ITEM(31, IOMUXC_BASE_ADDR + 0x5ac, 0x00000030)
MXC_DCD_ITEM(32, IOMUXC_BASE_ADDR + 0x5b4, 0x00000030)
MXC_DCD_ITEM(33, IOMUXC_BASE_ADDR + 0x528, 0x00000030)
MXC_DCD_ITEM(34, IOMUXC_BASE_ADDR + 0x520, 0x00000030)
MXC_DCD_ITEM(35, IOMUXC_BASE_ADDR + 0x514, 0x00000030)
MXC_DCD_ITEM(36, IOMUXC_BASE_ADDR + 0x510, 0x00000030)
MXC_DCD_ITEM(37, IOMUXC_BASE_ADDR + 0x5bc, 0x00000030)
MXC_DCD_ITEM(38, IOMUXC_BASE_ADDR + 0x5c4, 0x00000030)

MXC_DCD_ITEM(39, MMDC_P0_BASE_ADDR + 0x800, 0xA1390003)

MXC_DCD_ITEM(40, MMDC_P0_BASE_ADDR + 0x80c, 0x001F001F)
MXC_DCD_ITEM(41, MMDC_P0_BASE_ADDR + 0x810, 0x001F001F)
MXC_DCD_ITEM(42, MMDC_P1_BASE_ADDR + 0x80c, 0x001F001F)
MXC_DCD_ITEM(43, MMDC_P1_BASE_ADDR + 0x810, 0x001F001F)

MXC_DCD_ITEM(44, MMDC_P0_BASE_ADDR + 0x83c, 0x4333033F)
MXC_DCD_ITEM(45, MMDC_P0_BASE_ADDR + 0x840, 0x032C031D)
MXC_DCD_ITEM(46, MMDC_P1_BASE_ADDR + 0x83c, 0x43200332)
MXC_DCD_ITEM(47, MMDC_P1_BASE_ADDR + 0x840, 0x031A026A)
MXC_DCD_ITEM(48, MMDC_P0_BASE_ADDR + 0x848, 0x4D464746)
MXC_DCD_ITEM(49, MMDC_P1_BASE_ADDR + 0x848, 0x47453F4D)
MXC_DCD_ITEM(50, MMDC_P0_BASE_ADDR + 0x850, 0x3E434440)
MXC_DCD_ITEM(51, MMDC_P1_BASE_ADDR + 0x850, 0x47384839)

MXC_DCD_ITEM(52, MMDC_P0_BASE_ADDR + 0x81c, 0x33333333)
MXC_DCD_ITEM(53, MMDC_P0_BASE_ADDR + 0x820, 0x33333333)
MXC_DCD_ITEM(54, MMDC_P0_BASE_ADDR + 0x824, 0x33333333)
MXC_DCD_ITEM(55, MMDC_P0_BASE_ADDR + 0x828, 0x33333333)
MXC_DCD_ITEM(56, MMDC_P1_BASE_ADDR + 0x81c, 0x33333333)
MXC_DCD_ITEM(57, MMDC_P1_BASE_ADDR + 0x820, 0x33333333)
MXC_DCD_ITEM(58, MMDC_P1_BASE_ADDR + 0x824, 0x33333333)
MXC_DCD_ITEM(59, MMDC_P1_BASE_ADDR + 0x828, 0x33333333)

MXC_DCD_ITEM(60, MMDC_P0_BASE_ADDR + 0x8b8, 0x00000800)
MXC_DCD_ITEM(61, MMDC_P1_BASE_ADDR + 0x8b8, 0x00000800)

MXC_DCD_ITEM(62, MMDC_P0_BASE_ADDR + 0x004, 0x00020036)
MXC_DCD_ITEM(63, MMDC_P0_BASE_ADDR + 0x008, 0x09444040)
MXC_DCD_ITEM(64, MMDC_P0_BASE_ADDR + 0x00c, 0x555A7975)
MXC_DCD_ITEM(65, MMDC_P0_BASE_ADDR + 0x010, 0xFF538F64)
MXC_DCD_ITEM(66, MMDC_P0_BASE_ADDR + 0x014, 0x01FF00DB)
MXC_DCD_ITEM(67, MMDC_P0_BASE_ADDR + 0x018, 0x00001740)

MXC_DCD_ITEM(68, MMDC_P0_BASE_ADDR + 0x01c, 0x00008000)
MXC_DCD_ITEM(69, MMDC_P0_BASE_ADDR + 0x02c, 0x000026D2)
MXC_DCD_ITEM(70, MMDC_P0_BASE_ADDR + 0x030, 0x005A1023)
MXC_DCD_ITEM(71, MMDC_P0_BASE_ADDR + 0x040, 0x00000027)

MXC_DCD_ITEM(72, MMDC_P0_BASE_ADDR + 0x000, 0x831A0000)

MXC_DCD_ITEM(73, MMDC_P0_BASE_ADDR + 0x01c, 0x04088032)
MXC_DCD_ITEM(74, MMDC_P0_BASE_ADDR + 0x01c, 0x00008033)
MXC_DCD_ITEM(75, MMDC_P0_BASE_ADDR + 0x01c, 0x00048031)
MXC_DCD_ITEM(76, MMDC_P0_BASE_ADDR + 0x01c, 0x09408030)
MXC_DCD_ITEM(77, MMDC_P0_BASE_ADDR + 0x01c, 0x04008040)

MXC_DCD_ITEM(78, MMDC_P0_BASE_ADDR + 0x020, 0x00005800)

MXC_DCD_ITEM(79, MMDC_P0_BASE_ADDR + 0x818, 0x00011117)
MXC_DCD_ITEM(80, MMDC_P1_BASE_ADDR + 0x818, 0x00011117)

MXC_DCD_ITEM(81, MMDC_P0_BASE_ADDR + 0x004, 0x00025576)
MXC_DCD_ITEM(82, MMDC_P0_BASE_ADDR + 0x404, 0x00011006)
MXC_DCD_ITEM(83, MMDC_P0_BASE_ADDR + 0x01c, 0x00000000)

#endif

#endif
```

查看u-boot.lds可以了解uboot编译之后的布局

```
OUTPUT_FORMAT("elf32-littlearm", "elf32-littlearm", "elf32-littlearm")
OUTPUT_ARCH(arm)
ENTRY(_start)
SECTIONS
{
	. = 0x00000000;

	. = ALIGN(4);
	.text	   :
	{
	  /* WARNING - the following is hand-optimized to fit within	*/
	  /* the sector layout of our flash chips!	XXX FIXME XXX	*/
	  board/freescale/mx6q_sabresd/flash_header.o	(.text.flasheader)		# flash_header.S
	  cpu/arm_cortexa8/start.o												# start.S 
	  board/freescale/mx6q_sabresd/libmx6q_sabresd.a	(.text)
	  lib_arm/libarm.a		(.text)
	  net/libnet.a			(.text)
	  drivers/mtd/libmtd.a		(.text)
	  drivers/mmc/libmmc.a		(.text)

	  . = DEFINED(env_offset) ? env_offset : .;
	  common/env_embedded.o(.text)

	  *(.text)
	}

	. = ALIGN(4);
	.rodata : { *(SORT_BY_ALIGNMENT(SORT_BY_NAME(.rodata*))) }

	. = ALIGN(4);
	.data : { *(.data) }

	. = ALIGN(4);
	.got : { *(.got) }

	. = .;
	__u_boot_cmd_start = .;
	.u_boot_cmd : { *(.u_boot_cmd) }
	__u_boot_cmd_end = .;

	. = ALIGN(4);
	_end_of_copy = .; /* end_of ROM copy code here */

	/* Extend to align to 0x1000, then put the Hab Data */
	. = ALIGN(0x1000);
	__hab_data = .;
	. = . + 0x2000;
	__data_enc_key = .;
	/* actually, only 64bytes are needed, but this generates
		a size multiple of 512bytes, which is optimal for SD boot */
	. = . + 0x200;
	__hab_data_end = .;
	/* End of Hab Data, Place it before BSS section */

	__bss_start = .;
	.bss : { *(.bss) }
	_end = .;
}
```


查看编译uboot生成的System.map也能够知道大概的分布情况。

System.map

```
# 对应flash_header.S中的内容，地址连续
27800400 t ivt_header		# 首先是ivt header
27800404 t app_code_jump_v	# _start
27800408 t reserv1
2780040c t dcd_ptr
27800410 t boot_data_ptr
27800414 t self_ptr
27800418 t app_code_csf
2780041c t reserv2
27800420 t boot_data
27800424 t image_len
27800428 t plugin
2780042c t dcd_hdr
27800430 t write_dcd_cmd
# 对应 flash_header.S中的内容，每个dcd_node大小8字节。
# 前4个字节存放寄存器的地址，后4个字节设置寄存器的值。
27800434 t dcd_node_1		
2780043c t dcd_node_2
27800444 t dcd_node_3
2780044c t dcd_node_4
27800454 t dcd_node_5
2780045c t dcd_node_6
27800464 t dcd_node_7
2780046c t dcd_node_8
27800474 t dcd_node_9
2780047c t dcd_node_10
27800484 t dcd_node_11
2780048c t dcd_node_12
27800494 t dcd_node_13
2780049c t dcd_node_14
278004a4 t dcd_node_15
278004ac t dcd_node_16
278004b4 t dcd_node_17
278004bc t dcd_node_18
278004c4 t dcd_node_19

......

# uboot, start函数。对应cpu/arm_cortexa8/start.S中的内容
278006e0 T _start		
27800700 t _undefined_instruction
27800704 t _software_interrupt
27800708 t _prefetch_abort
2780070c t _data_abort
27800710 t _not_used
27800714 t _irq
27800718 t _fiq
2780071c t _pad
27800720 T _end_vect
27800720 t _TEXT_BASE
27800724 T _armboot_start
27800728 T _bss_start
2780072c T _bss_end
27800730 t reset
...
```

imx6开启启动之后，运行板子上的ROM程序。ROM确定启动的设备，进行一些初始化，然后读取IVT，进行寄存器初始化,最后运行uboot/cpu/arm_cortexa8/start.S中的_start函数。

IVT中记录了寄存器的地址和要设置的值，以及uboot的_start函数。

初始化完成之后，最后运行uboot/cpu/arm_cortexa8/start.S中的 _start函数，进行uboot的初始化。

start.S

```
.global _start
_start: b   reset
    ldr pc, _undefined_instruction
    ldr pc, _software_interrupt
    ldr pc, _prefetch_abort
    ldr pc, _data_abort
    ldr pc, _not_used
    ldr pc, _irq
    ldr pc, _fiq
```

Tony Liu

2016-12-20, Shenzhen