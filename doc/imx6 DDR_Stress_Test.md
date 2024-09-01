在调试DDR的时候，有时候需要更改参数。今天发现NXP提供了DDR Stress Test工具，用于DDR参数的校准。

#### 参考链接

http://blog.csdn.net/qq405180763/article/details/44977449

http://www.imx6rex.com/software/how-to-run-ddr3-calibration-on-imx6/

#### 验证

根据自己的需要更新选中imx6dl的文件，运行。

```		
C:\Users\Tony\Desktop\DDR_Stress_Tester_V1.0.2\Binary>DDR_Stress_Tester.exe -t mx6x -df scripts\MX6_series_boards\SabreSD\RevC_and_RevB\MX6DL\MX6DL_SabreSD_DDR3_register_programming_aid_v1.5.inc
MX6DL opened.
HAB_TYPE: DEVELOP
Image loading...
download Image to IRAM OK

Re-open MX6x device.
Running DDR test..., press "ESC" key to exit.


******************************
    DDR Stress Test (1.0.2) for MX6DL
    Build: Dec 10 2013, 12:31:47
    Freescale Semiconductor, Inc.
******************************

=======DDR configuration==========
BOOT_CFG3[5-4]: 0x00, Single DDR channel.
DDR type is DDR3
Data width: 64, bank num: 8
Row size: 14, col size: 10
Chip select CSD0 is used
Density per chip select: 1024MB
==================================


What ARM core speed would you like to run?
Type 0 for 650MHz, 1 for 800MHz, 2 for 1GHz
  ARM set to 800MHz

Please select the DDR density per chip select (in bytes) on the board
Type 0 for 2GB; 1 for 1GB; 2 for 512MB; 3 for 256MB; 4 for 128MB; 5 for 64MB; 6 for 32MB
For maximum supported density (4GB), we can only access up to 3.75GB.  Type 9 to select this
  DDR density selected (MB): 256


Calibration will run at DDR frequency 400MHz. Type 'y' to continue.
If you want to run at other DDR frequency. Type 'n'
  DDR Freq: 396 MHz

Would you like to run the write leveling calibration? (y/n)
  Please enter the MR1 value on the initilization script
  This will be re-programmed into MR1 after write leveling calibration
  Enter as a 4-digit HEX value, example 0004, then hit enter
0004 You have entered: 0x0004
Start write leveling calibration
Write leveling calibration completed
MMDC_MPWLDECTRL0 ch0 after write level cal: 0x0047004A
MMDC_MPWLDECTRL1 ch0 after write level cal: 0x003F0045
MMDC_MPWLDECTRL0 ch1 after write level cal: 0x002D0030
MMDC_MPWLDECTRL1 ch1 after write level cal: 0x002D0047

Would you like to run the DQS gating, read/write delay calibration? (y/n)
Starting DQS gating calibration...
. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
. . .
BYTE 0:
        Start:           HC=0x00 ABS=0x08
        End:             HC=0x03 ABS=0x44
        Mean:            HC=0x01 ABS=0x65
        End-0.5*tCK:     HC=0x02 ABS=0x44
        Final:           HC=0x02 ABS=0x44
BYTE 1:
        Start:           HC=0x01 ABS=0x04
        End:             HC=0x03 ABS=0x48
        Mean:            HC=0x02 ABS=0x26
        End-0.5*tCK:     HC=0x02 ABS=0x48
        Final:           HC=0x02 ABS=0x48
BYTE 2:
        Start:           HC=0x00 ABS=0x78
        End:             HC=0x03 ABS=0x34
        Mean:            HC=0x02 ABS=0x16
        End-0.5*tCK:     HC=0x02 ABS=0x34
        Final:           HC=0x02 ABS=0x34
BYTE 3:
        Start:           HC=0x00 ABS=0x78
        End:             HC=0x03 ABS=0x34
        Mean:            HC=0x02 ABS=0x16
        End-0.5*tCK:     HC=0x02 ABS=0x34
        Final:           HC=0x02 ABS=0x34
BYTE 4:
        Start:           HC=0x00 ABS=0x00
        End:             HC=0x03 ABS=0x34
        Mean:            HC=0x01 ABS=0x59
        End-0.5*tCK:     HC=0x02 ABS=0x34
        Final:           HC=0x02 ABS=0x34
BYTE 5:
        Start:           HC=0x00 ABS=0x6C
        End:             HC=0x03 ABS=0x2C
        Mean:            HC=0x02 ABS=0x0C
        End-0.5*tCK:     HC=0x02 ABS=0x2C
        Final:           HC=0x02 ABS=0x2C
BYTE 6:
        Start:           HC=0x00 ABS=0x64
        End:             HC=0x03 ABS=0x24
        Mean:            HC=0x02 ABS=0x04
        End-0.5*tCK:     HC=0x02 ABS=0x24
        Final:           HC=0x02 ABS=0x24
BYTE 7:
        Start:           HC=0x00 ABS=0x58
        End:             HC=0x03 ABS=0x1C
        Mean:            HC=0x01 ABS=0x79
        End-0.5*tCK:     HC=0x02 ABS=0x1C
        Final:           HC=0x02 ABS=0x1C

DQS calibration MMDC0 MPDGCTRL0 = 0x42480244, MPDGCTRL1 = 0x02340234

DQS calibration MMDC1 MPDGCTRL0 = 0x422C0234, MPDGCTRL1 = 0x021C0224

Note: Array result[] holds the DRAM test result of each byte.
      0: test pass.  1: test fail
      4 bits respresent the result of 1 byte.
      result 00000001:byte 0 fail.
      result 00000011:byte 0, 1 fail.

Starting Read calibration...

ABS_OFFSET=0x00000000   result[00]=0x11111111
ABS_OFFSET=0x04040404   result[01]=0x11111111
ABS_OFFSET=0x08080808   result[02]=0x11111111
ABS_OFFSET=0x0C0C0C0C   result[03]=0x11111111
ABS_OFFSET=0x10101010   result[04]=0x11111111
ABS_OFFSET=0x14141414   result[05]=0x11111111
ABS_OFFSET=0x18181818   result[06]=0x11111111
ABS_OFFSET=0x1C1C1C1C   result[07]=0x01111111
ABS_OFFSET=0x20202020   result[08]=0x00100110
ABS_OFFSET=0x24242424   result[09]=0x00000000
ABS_OFFSET=0x28282828   result[0A]=0x00000000
ABS_OFFSET=0x2C2C2C2C   result[0B]=0x00000000
ABS_OFFSET=0x30303030   result[0C]=0x00000000
ABS_OFFSET=0x34343434   result[0D]=0x00000000
ABS_OFFSET=0x38383838   result[0E]=0x00000000
ABS_OFFSET=0x3C3C3C3C   result[0F]=0x00000000
ABS_OFFSET=0x40404040   result[10]=0x00000000
ABS_OFFSET=0x44444444   result[11]=0x00000000
ABS_OFFSET=0x48484848   result[12]=0x00000000
ABS_OFFSET=0x4C4C4C4C   result[13]=0x00000000
ABS_OFFSET=0x50505050   result[14]=0x00000000
ABS_OFFSET=0x54545454   result[15]=0x00000000
ABS_OFFSET=0x58585858   result[16]=0x00000000
ABS_OFFSET=0x5C5C5C5C   result[17]=0x00000000
ABS_OFFSET=0x60606060   result[18]=0x00000000
ABS_OFFSET=0x64646464   result[19]=0x00000000
ABS_OFFSET=0x68686868   result[1A]=0x00000000
ABS_OFFSET=0x6C6C6C6C   result[1B]=0x00010000
ABS_OFFSET=0x70707070   result[1C]=0x11111111
ABS_OFFSET=0x74747474   result[1D]=0x11111111
ABS_OFFSET=0x78787878   result[1E]=0x11111111
ABS_OFFSET=0x7C7C7C7C   result[1F]=0x11111111

MMDC0 MPRDDLCTL = 0x46484846, MMDC1 MPRDDLCTL = 0x44464844

Starting Write calibration...

ABS_OFFSET=0x00000000   result[00]=0x10101100
ABS_OFFSET=0x04040404   result[01]=0x10000100
ABS_OFFSET=0x08080808   result[02]=0x00000100
ABS_OFFSET=0x0C0C0C0C   result[03]=0x00000000
ABS_OFFSET=0x10101010   result[04]=0x00000000
ABS_OFFSET=0x14141414   result[05]=0x00000000
ABS_OFFSET=0x18181818   result[06]=0x00000000
ABS_OFFSET=0x1C1C1C1C   result[07]=0x00000000
ABS_OFFSET=0x20202020   result[08]=0x00000000
ABS_OFFSET=0x24242424   result[09]=0x00000000
ABS_OFFSET=0x28282828   result[0A]=0x00000000
ABS_OFFSET=0x2C2C2C2C   result[0B]=0x00000000
ABS_OFFSET=0x30303030   result[0C]=0x00000000
ABS_OFFSET=0x34343434   result[0D]=0x00000000
ABS_OFFSET=0x38383838   result[0E]=0x00000000
ABS_OFFSET=0x3C3C3C3C   result[0F]=0x00000000
ABS_OFFSET=0x40404040   result[10]=0x00000000
ABS_OFFSET=0x44444444   result[11]=0x00000000
ABS_OFFSET=0x48484848   result[12]=0x00000000
ABS_OFFSET=0x4C4C4C4C   result[13]=0x00000100
ABS_OFFSET=0x50505050   result[14]=0x00000100
ABS_OFFSET=0x54545454   result[15]=0x00000100
ABS_OFFSET=0x58585858   result[16]=0x00010100
ABS_OFFSET=0x5C5C5C5C   result[17]=0x00010100
ABS_OFFSET=0x60606060   result[18]=0x00110110
ABS_OFFSET=0x64646464   result[19]=0x10110111
ABS_OFFSET=0x68686868   result[1A]=0x11110111
ABS_OFFSET=0x6C6C6C6C   result[1B]=0x11111111
ABS_OFFSET=0x70707070   result[1C]=0x11111111
ABS_OFFSET=0x74747474   result[1D]=0x11111111
ABS_OFFSET=0x78787878   result[1E]=0x11111111
ABS_OFFSET=0x7C7C7C7C   result[1F]=0x11111111

MMDC0 MPWRDLCTL = 0x362A2E30,MMDC1 MPWRDLCTL = 0x3432302A


   MMDC registers updated from calibration

   Read DQS Gating calibration
   MPDGCTRL0 PHY0 (0x021b083c) = 0x42480244
   MPDGCTRL1 PHY0 (0x021b0840) = 0x02340234
   MPDGCTRL0 PHY1 (0x021b483c) = 0x422C0234
   MPDGCTRL1 PHY1 (0x021b4840) = 0x021C0224

   Read calibration
   MPRDDLCTL PHY0 (0x021b0848) = 0x46484846
   MPRDDLCTL PHY1 (0x021b4848) = 0x44464844

   Write calibration
   MPWRDLCTL PHY0 (0x021b0850) = 0x362A2E30
   MPWRDLCTL PHY1 (0x021b4850) = 0x3432302A


The DDR stress test can run with an incrementing frequency or at a static freq
To run at a static freq, simply set the start freq and end freq to the same value
Would you like to run the DDR Stress Test (y/n)?
^C
C:\Users\Tony\Desktop\DDR_Stress_Tester_V1.0.2\Binary>
```		

将参数更新到MX6DL_SabreSD_DDR3_register_programming_aid_v1.5.inc,重复运行多次。

数据更新到uboot/board/freescale/mx6q_sabresd/flash_header.S中。详情请看本文的参考链接。

```
   MMDC registers updated from calibration

   Read DQS Gating calibration
   MPDGCTRL0 PHY0 (0x021b083c) = 0x42480244
   MPDGCTRL1 PHY0 (0x021b0840) = 0x02340234
   MPDGCTRL0 PHY1 (0x021b483c) = 0x422C0234
   MPDGCTRL1 PHY1 (0x021b4840) = 0x021C0224

   Read calibration
   MPRDDLCTL PHY0 (0x021b0848) = 0x46484846
   MPRDDLCTL PHY1 (0x021b4848) = 0x44464844

   Write calibration
   MPWRDLCTL PHY0 (0x021b0850) = 0x362A2E30
   MPWRDLCTL PHY1 (0x021b4850) = 0x3432302A
```

Tony Liu

2016-12-20, Shenzhen