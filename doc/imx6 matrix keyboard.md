imx6需要添加4x4的矩阵键盘。本文记录添加方法。

#### 参考链接

http://processors.wiki.ti.com/index.php/TI-Android-JB-PortingGuide

http://blog.sina.com.cn/s/blog_54aa47930102vesb.html

http://blog.csdn.net/shell_albert/article/details/46801091

http://www.kikikoo.com/uid-20768928-id-5084287.html

http://blog.csdn.net/shell_albert/article/details/46618655

#### 添加驱动

驱动源码driver/input/keyboard/matrix_keypad.c
```
CONFIG_KEYBOARD_MATRIX:                                       
                                                             
    Enable support for GPIO driven matrix keypad.     
                                                      
    To compile this driver as a module, choose M here: the 
    module will be called matrix_keypad.          
                                                  
    Symbol: KEYBOARD_MATRIX [=y]                  
    Type  : tristate                              
    Prompt: GPIO driven matrix keypad support     
      Defined at drivers/input/keyboard/Kconfig:224  
      Depends on: !S390 && INPUT [=y] && INPUT_KEYBOARD [=y] && GENERIC_GPIO [=y]    
      Location:                                                                      
        -> Device Drivers                                                            
          -> Input device support                                                    
            -> Generic input layer (needed for keyboard, mouse, ...) (INPUT [=y])    
              -> Keyboards (INPUT_KEYBOARD [=y]) 
```

#### 添加设备

vi arch/arm/mach-mx6/board-mx6q_sabresd.c
```
// COL
#define SABRESD_KEY0     IMX_GPIO_NR(4, 6)
#define SABRESD_KEY1     IMX_GPIO_NR(4, 8)
#define SABRESD_KEY2     IMX_GPIO_NR(4, 10)
#define SABRESD_KEY3     IMX_GPIO_NR(4, 12)
// ROW
#define SABRESD_KEY4     IMX_GPIO_NR(4, 7)
#define SABRESD_KEY5     IMX_GPIO_NR(4, 9)
#define SABRESD_KEY6     IMX_GPIO_NR(4, 11)
#define SABRESD_KEY7     IMX_GPIO_NR(4, 13)

static const uint32_t sabresd_keymap[] = {
    KEY(0, 0, KEY_ESC),
    KEY(0, 1, KEY_0),
    KEY(0, 2, KEY_1),
    KEY(0, 3, KEY_OK),
    KEY(1, 0, KEY_2),
    KEY(1, 1, KEY_3),
    KEY(1, 2, KEY_4),
    KEY(1, 3, KEY_5),
    KEY(2, 0, KEY_6),
    KEY(2, 1, KEY_7),
    KEY(2, 2, KEY_8),
    KEY(2, 3, KEY_9),
    KEY(3, 0, KEY_F1),
    KEY(3, 1, KEY_0),
    KEY(3, 2, KEY_F2),
    KEY(3, 3, KEY_F3),
};

static const struct matrix_keymap_data sabresd_keymap_data = {
    .keymap  = sabresd_keymap,
    .keymap_size = ARRAY_SIZE(sabresd_keymap),
};

static const unsigned int sabresd_keypad_cols[] = {
    SABRESD_KEY0,
    SABRESD_KEY1,
    SABRESD_KEY2,
    SABRESD_KEY3,
};
 
static const unsigned int sabresd_keypad_rows[] = {
    SABRESD_KEY4,
    SABRESD_KEY5,
    SABRESD_KEY6,
    SABRESD_KEY7,
};
 
static struct matrix_keypad_platform_data sabresd_kpd_pdata = {
    .keymap_data = &sabresd_keymap_data,
    .col_gpios = sabresd_keypad_cols,
    .row_gpios = sabresd_keypad_rows,
    .num_col_gpios = ARRAY_SIZE(sabresd_keypad_cols),
    .num_row_gpios = ARRAY_SIZE(sabresd_keypad_rows),
    .col_scan_delay_us = 10,
    .debounce_ms  = 10,
    .wakeup   = 1,
    .active_low  = 1,
    //.active_low  = 0,
};
 
static struct platform_device sabresd_keypad = {
    .name  = "matrix-keypad",
    .id  = -1,
    .dev  = {
     .platform_data = &sabresd_kpd_pdata,
    },
};

再添加初始化函数
static void __init mx6_sabresd_board_init(void)
{
	......
	platform_device_register(&sabresd_keypad)
	......
}

```

#### 查看效果

```
root@android:/data # cat /proc/bus/input/devices                               
I: Bus=0019 Vendor=0001 Product=0001 Version=0100
N: Name="gpio-keys"
P: Phys=gpio-keys/input0
S: Sysfs=/devices/platform/gpio-keys/input/input0
U: Uniq=
H: Handlers=event0 
B: PROP=0
B: EV=3
B: KEY=100000 0 0 0

I: Bus=0019 Vendor=0000 Product=0000 Version=0000
N: Name="matrix-keypad"
P: Phys=
S: Sysfs=/devices/platform/matrix-keypad/input/input1
U: Uniq=
H: Handlers=event1 
B: PROP=0
B: EV=100013
B: KEY=1 0 0 0 0 0 0 0 0 0 38000000 ffe
B: MSC=10

I: Bus=0003 Vendor=09da Product=c10a Version=0110
N: Name="A4Tech USB Mouse"
P: Phys=usb-fsl-ehci.1-1.3/input0
S: Sysfs=/devices/platform/fsl-ehci.1/usb2/2-1/2-1.3/2-1.3:1.0/input/input2
U: Uniq=
H: Handlers=mouse0 event2 
B: PROP=0
B: EV=1f
B: KEY=ff0000 0 0 0 0 0 0 0 0
B: REL=303
B: ABS=1f00 0
B: MSC=10

root@android:/data # getevent
could not get driver version for /dev/input/mice, Not a typewriter
add device 1: /dev/input/event2
  name:     "A4Tech USB Mouse"
could not get driver version for /dev/input/mouse0, Not a typewriter
add device 2: /dev/input/event1
  name:     "matrix-keypad"				# matrix-keypad
add device 3: /dev/input/event0
  name:     "gpio-keys"					# gpio-keys```
```

Tony Liu

2016-12-30, Shenzhen