imx6 usb的host和slave配置，配置之后,安装gadget模块，就能够在host和slave之间切换。

参考文档：

i.MX 6Dual/6Quad Linux Reference Manual

http://www.cnblogs.com/helloworldtoyou/p/6181860.html

将内核.config文件中下列选项选中（Y），或者编译成模块编译成M。

以下是摘自i.MX 6Dual/6Quad Linux Reference Manual中的内容

Chapter 34 ARC USB Driver
```
• CONFIG_USB-Build support for USB
• CONFIG_USB_EHCI_HCD-Build support for USB host driver. In menuconfig, this
option is available under Device drivers > USB support > EHCI HCD (USB 2.0)
support.
By default, this option is Y.
• CONFIG_USB_EHCI_ARC-Build support for selecting the ARC EHCI host. In
menuconfig, this option is available under Device drivers > USB support > Support
for Freescale controller.
By default, this option is Y.
• CONFIG_USB_EHCI_ARC_OTG-Build support for selecting the ARC EHCI OTG
host. In menuconfig, this option is available under
Device drivers > USB support > EHCI HCD (USB 2.0) support > Support for DR
host port on Freescale controller.
By default, this option is Y.
• CONFIG_USB_EHCI_ARC_HSIC Freescale HSIC USB Host Controller
By default, this option is N.
• CONFIG_USB_EHCI_ROOT_HUB_TT-Some EHCI chips have vendor-specific
extensions to integrate transaction translators, so that no OHCI or UHCI companion
controller is needed. In menuconfig this option is available under
Device drivers > USB support > Root Hub Transaction Translators.
By default, this option is Y selected by USB_EHCI_ARC && USB_EHCI_HCD.
• CONFIG_USB_STORAGE-Build support for USB mass storage devices. In
menuconfig this option is available under
Device drivers > USB support > USB Mass Storage support.
By default, this option is Y.
• CONFIG_USB_HID-Build support for all USB HID devices. In menuconfig this
option is available under

Device drivers > HID Devices > USB Human Interface Device (full HID) support.
By default, this option is Y.
• CONFIG_USB_GADGET-Build support for USB gadget. In menuconfig, this option
is available under
Device drivers > USB support > USB Gadget Support.
By default, this option is M.
• CONFIG_USB_GADGET_ARC-Build support for ARC USB gadget. In
menuconfig, this option is available under
Device drivers > USB support > USB Gadget Support > USB Peripheral Controller
(Freescale USB Device Controller).
By default, this option is Y.
• CONFIG_IMX_USB_CHARGER Freescale i.MX 6 USB Charger Detection
By default, this option is N.
• CONFIG_USB_OTG-OTG Support, support dual role with ID pin detection.
By default, this option is Y.
• CONFIG_MXC_OTG-USB OTG pin detect support for Freescale USB OTG
Controller
By default, this option is Y.
• CONFIG_USB_ETH-Build support for Ethernet gadget. In menuconfig, this option
is available under
Device drivers > USB support > USB Gadget Support > Ethernet Gadget (with CDC
Ethernet Support).
By default, this option is M.
• CONFIG_USB_ETH_RNDIS-Build support for Ethernet RNDIS protocol. In
menuconfig, this option is available under
Device drivers > USB support > USB Gadget Support > Ethernet Gadget (with CDC
Ethernet Support) > RNDIS support.
By default, this option is Y.
• CONFIG_USB_FILE_STORAGE-Build support for Mass Storage gadget. In
menuconfig, this option is available under
Device drivers > USB support > USB Gadget Support > File-backed Storage Gadget.
By default, this option is M.
• CONFIG_USB_G_SERIAL-Build support for ACM gadget. In menuconfig, this
option is available under
Device drivers > USB support > USB Gadget Support > Serial Gadget (with CDC
ACM support).
By default, this option is M.
```

Tony Liu

2016-12-15， ShenZhen