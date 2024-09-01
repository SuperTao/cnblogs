最近需要使用到power button按键，linux中有gpio keys的机制，只要注册即可。

#### device注册

arch/arm/mach-mx6/board-mx6q_sabresd.c
```
#define SABRESD_POWER_OFF	IMX_GPIO_NR(3, 29)

static struct gpio_keys_button new_sabresd_buttons[] = {
	GPIO_BUTTON(SABRESD_VOLUME_UP, KEY_VOLUMEUP, 1, "volume-up", 0, 1),
	GPIO_BUTTON(SABRESD_VOLUME_DN, KEY_VOLUMEDOWN, 1, "volume-down", 0, 1),
	GPIO_BUTTON(SABRESD_POWER_OFF, KEY_POWER, 1, "power-key", 1, 1),
};

static struct gpio_keys_platform_data new_sabresd_button_data = {
	.buttons	= new_sabresd_buttons,
	.nbuttons	= ARRAY_SIZE(new_sabresd_buttons),
};

static struct platform_device sabresd_button_device = {
	.name		= "gpio-keys",
	.id		= -1,
	.num_resources  = 0,
};

static void __init imx6q_add_device_buttons(void)
{
	/* fix me */
	/* For new sabresd(RevB4 ane above) change the
	 * ONOFF key(SW1) design, the SW1 now connect
	 * to GPIO_3_29, it can be use as a general power
	 * key that Android reuired. But those old sabresd
	 * such as RevB or older could not support this
	 * change, so it needs a way to distinguish different
	 * boards. Before board id/rev are defined cleary,
	 * there is a simple way to achive this, that is using
	 * SOC revison to identify differnt board revison.
	 *
	 * With the new sabresd change and SW mapping the
	 * SW1 as power key, below function related to power
	 * key are OK on new sabresd board(B4 or above).
	 * 	1 Act as power button to power on the device when device is power off
	 * 	2 Act as power button to power on the device(need keep press SW1 >5s)
	 *	3 Act as power key to let device suspend/resume
	 *	4 Act screenshort(hold power key and volume down key for 2s)
	 */
		platform_device_add_data(&sabresd_button_device,
				&new_sabresd_button_data,
				sizeof(new_sabresd_button_data));
	
	platform_device_register(&sabresd_button_device);
}


static void __init mx6_sabresd_board_init(void)
{
    ......
	imx6q_add_device_buttons();
    ......
}
```

#### driver

对应驱动代码

drivers/input/keyboard/gpio_keys.c


#### test

加载驱动之后，按住power key，却不能熄灭屏幕。由于是linux系统，与安卓有所不同，这一块的问题还没解决。

目前只是在文件系统层，使系统进入低功耗模式，power key就能够唤醒系统。不过这样每次都得手动操作才能使屏幕熄灭。

echo mem > /sys/power/state

echo standby > /sys/power/state

```
root@freescale /sys/power$ ll
-rw-r--r--    1 root     root          4096 Jan  1 00:15 device_suspend_time_threshold
-rw-r--r--    1 root     root          4096 Jan  1 00:15 pm_async
-rw-r--r--    1 root     root          4096 Jan  1 00:15 pm_test
-rw-r--r--    1 root     root          4096 Jan  1 00:15 state
-rw-r--r--    1 root     root          4096 Jan  1 00:15 wakeup_count
# 查看支持的模式
root@freescale /sys/power$ cat state 
standby mem
# 使cpu进入standby模式 (或者mem模式), 此时屏幕熄灭
root@freescale /sys/power$ echo standby > state
PM: Syncing filesystems ... done.
Freezing user space processes ... (elapsed 0.01 seconds) done.
Freezing remaining freezable tasks ... (elapsed 0.01 seconds) done.
Suspending console(s) (use no_console_suspend to debug)
# 按住power key，屏幕又重新点亮
add wake up source irq 103
PM: suspend of devices complete after 3.163 msecs
PM: suspend devices took 0.010 seconds
PM: late suspend of devices complete after 0.701 msecs
Disabling non-boot CPUs ...
CPU1: shutdown
Suspended for 0.000 seconds
Enabling non-boot CPUs ...
CPU1: Booted secondary processor
Calibrating delay loop (skipped) already calibrated this CPU
i.MXC CPU frequency driver
CPU1 is up
PM: early resume of devices complete after 0.497 msecs
imx-ipuv3 imx-ipuv3.0: IPU DMFC NORMAL mode: 1(0~1), 5B(4,5), 5F(6,7)
remove wake up source irq 103
usb 2-1: reset high speed USB device number 2 using fsl-ehci
usb 2-1.1: reset low speed USB device number 3 using fsl-ehci
PM: resume of devices complete after 1357.464 msecs
PM: resume devices took 1.360 seconds
Restarting tasks ... done.
```

Tony Liu

2016-12-3, Shenzhen