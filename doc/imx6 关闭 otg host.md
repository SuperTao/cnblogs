参考文档：

　　http://www.cnblogs.com/zengjfgit/p/4711336.html

```
make menuconfig
去掉Support for DR host port on Freescale controller    

    [*] Device Drivers --->
       [*] USB support --->
          [*]   Support for Freescale controller            
             [ ]     Support for DR host port on Freescale controller 

使用h查看帮组信息：
Support for DR host port on Freescale controller
CONFIG_USB_EHCI_ARC_OTG:                        
                                                            
Enable support for the USB OTG port in HS/FS Host mode.      

Symbol: USB_EHCI_ARC_OTG [=n]                               
Type  : boolean                                           
Prompt: Support for DR host port on Freescale controller   
  Defined at drivers/usb/host/Kconfig:80                 
  Depends on: USB_SUPPORT [=y] && USB_EHCI_ARC [=y]     
  Location:                                            
    -> Device Drivers                                 
      -> USB support (USB_SUPPORT [=y])              
        -> Support for Freescale controller (USB_EHCI_ARC [=y]) 
```

Tony Liu

2016-8-25, Shenzhen