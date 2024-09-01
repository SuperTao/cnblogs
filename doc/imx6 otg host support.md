本文记录添加imx6 otg host支持的过程。

#### 参考链接

http://www.cnblogs.com/helloworldtoyou/p/6108560.html

https://community.nxp.com/thread/328351

https://community.nxp.com/thread/305646

https://community.nxp.com/thread/387529

		   
#### 设备配置

* 配置引脚复用

这里遇到的问题主要是USB OTG POWER的引脚不能输出高电平，所以进行了一些修改。

arch/arm/mach-mx6/board-mx6dl_sabresd.h

```
#define MX6DL_ENET_PAD_CTRL_PD (PAD_CTL_PKE | PAD_CTL_PUE  |        \
        PAD_CTL_PUS_100K_DOWN | PAD_CTL_SPEED_MED |     \
        PAD_CTL_DSE_40ohm   | PAD_CTL_HYS)

MX6DL_PAD_KEY_COL4__USBOH3_USBOTG_OC,
// GPIO
IOMUX_PAD(0x0650, 0x0268, 5, 0x0000, 0, MX6DL_ENET_PAD_CTRL_PD), //USB OTG POWER
MX6DL_PAD_ENET_RX_ER__ANATOP_USBOTG_ID,
```

* 设备初始化

arch/arm/mach-mx6/board-mx6q_sabresd.c

```
#define SABRESD_USB_OTG_PWR IMX_GPIO_NR(4, 15)
#define SABRESD_USB_H1_PWR  IMX_GPIO_NR(7, 0)

static void imx6q_sabresd_usbotg_vbus(bool on)
{
    if (on) 
        gpio_set_value(SABRESD_USB_OTG_PWR, 1);
    else 
        gpio_set_value(SABRESD_USB_OTG_PWR, 0);
}

static void imx6q_sabresd_host1_vbus(bool on)
{
    if (on) 
        gpio_set_value(SABRESD_USB_H1_PWR, 1);
    else 
        gpio_set_value(SABRESD_USB_H1_PWR, 0);
}

static void __init imx6q_sabresd_init_usb(void)
{
    int ret = 0; 

    imx_otg_base = MX6_IO_ADDRESS(MX6Q_USB_OTG_BASE_ADDR);
    /* disable external charger detect,
     * or it will affect signal quality at dp .
     */
    ret = gpio_request(SABRESD_USB_OTG_PWR, "usb-pwr");
    if (ret) {
        pr_err("failed to get GPIO SABRESD_USB_OTG_PWR: %d\n",
            ret);
        return;
    }    
    gpio_direction_output(SABRESD_USB_OTG_PWR, 0);
	// USB host1 VBUS 在我这边得硬件上直连电源，所以就不需要控制
    /* keep USB host1 VBUS always on */
    /*   
    ret = gpio_request(SABRESD_USB_H1_PWR, "usb-h1-pwr");
    if (ret) {
        pr_err("failed to get GPIO SABRESD_USB_H1_PWR: %d\n",
            ret);
        return;
    }
    gpio_direction_output(SABRESD_USB_H1_PWR, 0);
    */ 
    
	if (board_is_mx6_reva())
        mxc_iomux_set_gpr_register(1, 13, 1, 1);
    else
        mxc_iomux_set_gpr_register(1, 13, 1, 0);

    mx6_set_otghost_vbus_func(imx6q_sabresd_usbotg_vbus);
    //mx6_set_host1_vbus_func(imx6q_sabresd_host1_vbus);
}
```

#### kernel config配置

这里只记录部分，配置完成，usb host就可以使用,不过slave还需要调试。

```
CONFIG_USB_EHCI_ARC_OTG=y
CONFIG_USB_EHCI_ARC_HSIC=y

CONFIG_USB_OTG=y 
CONFIG_MXC_OTG=y

CONFIG_USB_SUSPEND=n  
```

Tony Liu

2016-11-28, Shenzhen