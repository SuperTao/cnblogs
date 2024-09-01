imx6的mac地址总是固定的值，所以需要更改，采用的方法是在uboot中设置环境变量,之后在kernel中使用uboot中设置的mac地址的值。本文记录更改的过程。 

参考链接：

　　http://www.cnblogs.com/zengjfgit/p/5711304.html

#### uboot
```
lib_arm/board.c                                                                 
                                                                                
void start_armboot (void)                                                       
{                                                                               
    ...
    eth_initialize(gd->bd);                 ---------------------------+
    ...                                                                |        
};                                                                     |        
                                                                       |        
                                                                       |        
int eth_initialize(bd_t *bis)                            <-------------+        
{                                                                               
    ......
    /* Try board-specific initialization first.  If it fails or isn't           
     * present, try the cpu-specific initialization */                          
    if (board_eth_init(bis) < 0)                                                
        cpu_eth_init(bis);                          -------------+              
    ......                                                       |              
}                                                                |              
                                                                 |              
int cpu_eth_init(bd_t *bis)                             <--------+              
{                                                                               
    int rc = -ENODEV;                                                           
#if defined(CONFIG_MXC_FEC)                                                     
    rc = mxc_fec_initialize(bis);    ------+                                    
                                           |                                    
    /* Board level init */                 |                                    
    enet_board_init();                     |                                    
                                           |                                    
#endif                                     |                                    
    return rc;                             |                                    
}                                          |                                    
                                           |                                    
int mxc_fec_initialize(bd_t *bis)   <------+                                    
{                                                                               
    ......
        if (fec_get_hwaddr(dev, ethaddr) == 0) {                  ------+       
            printf("got MAC address from IIM: %pM\n", ethaddr);         |       
            memcpy(dev->enetaddr, ethaddr, 6);                          |       
            fec_set_hwaddr(dev);                                        |       
        }                                                               |       
    }                                                                   |       
                                                                        |       
    return 1;                                                           |       
}                                                                       |       
                                                                        |       
static int fec_get_hwaddr(struct eth_device *dev, unsigned char *mac)<--+       
{                                                                               
#ifdef CONFIG_GET_FEC_MAC_ADDR_FROM_IIM                                         
    fec_get_mac_addr(mac);                       -------+                       
    return 0;                                           |                       
#else                                                   |                       
    return -1;                                          |                       
#endif                                                  |                       
}                                                       |                       
                                                        |                       
#ifdef CONFIG_GET_FEC_MAC_ADDR_FROM_IIM                 |                       
int fec_get_mac_addr(unsigned char *mac)          <---- +                       
{                                                                               
#if 0                                                                           
    unsigned int value;                          //不读取寄存器中的值                               
                                                                                
    value = readl(OCOTP_BASE_ADDR + HW_OCOTP_MACn(0));                          
    mac[5] = value & 0xff;                                                      
    mac[4] = (value >> 8) & 0xff;                                               
    mac[3] = (value >> 16) & 0xff;                                              
    mac[2] = (value >> 24) & 0xff;                                              
    value = readl(OCOTP_BASE_ADDR + HW_OCOTP_MACn(1));                          
    mac[1] = value & 0xff;                                                      
    mac[0] = (value >> 8) & 0xff;                                               
#else                                                                           
    eth_getenv_enetaddr("ethaddr", mac);          //使用环境变量设置的mac地址                              
#endif                                                                          
    return 0;                                                                   
}                                                                               
```

#### kernel

1.设备

```
arch/arm/mach-mx6/board-mx6q_sabresd.c
MACHINE_START(MX6Q_SABRESD, "Freescale i.MX 6Quad/DualLite/Solo Sabre-SD Board")
    /* Maintainer: Freescale Semiconductor, Inc. */
    .boot_params = MX6_PHYS_OFFSET + 0x100,
    .fixup = fixup_mxc_board,
    .map_io = mx6_map_io,
    .init_irq = mx6_init_irq,
    .init_machine = mx6_sabresd_board_init,
    .timer = &mx6_sabresd_timer,
    .reserve = mx6q_sabresd_reserve,
MACHINE_END

static void __init mx6_sabresd_board_init(void)
{
    ......
    imx6_init_fec(fec_data);
    ......
}

arch/arm/mach-mx6/mx6_fec.c
void __init imx6_init_fec(struct fec_platform_data fec_data)
{
    //读取寄存器中的mac地址的值
    fec_get_mac_addr(fec_data.mac);
    //这里就已经随机生成了mac地址，在驱动中就不会再次设置，所以注释了
    //设备注册在驱动注册之前，先不要自动生成mac地址，由驱动来设置
//  if (!is_valid_ether_addr(fec_data.mac))
//      random_ether_addr(fec_data.mac);
    if (cpu_is_mx6sl())
        imx6sl_add_fec(&fec_data);
    else
        imx6q_add_fec(&fec_data);
}
```

2.驱动

```
drivers/net/fec.c
//驱动设置mac地址
static void __inline__ fec_get_mac(struct net_device *ndev)
{
	struct fec_enet_private *fep = netdev_priv(ndev);
	struct fec_platform_data *pdata = fep->pdev->dev.platform_data;
	unsigned char *iap, tmpaddr[ETH_ALEN];

	/*
	 * try to get mac address in following order:
	 *
	 * 1) module parameter via kernel command line in form
	 *    fec.macaddr=0x00,0x04,0x9f,0x01,0x30,0xe0
	 */

	iap = macaddr;
    // 检查是否合法，不合法就通过flash设置
	/*
	 * 2) from flash or fuse (via platform data)
	 */
	if (!is_valid_ether_addr(iap)) {
#ifdef CONFIG_M5272
		if (FEC_FLASHMAC)
			iap = (unsigned char *)FEC_FLASHMAC;
#else
		if (pdata)
			memcpy(iap, pdata->mac, ETH_ALEN);
#endif
	}
    // 检查是否合法，不合法就通过bootloader设置
	/*
	 * 3) FEC mac registers set by bootloader
	 */
	if (!is_valid_ether_addr(iap)) {
		*((unsigned long *) &tmpaddr[0]) =
			be32_to_cpu(readl(fep->hwp + FEC_ADDR_LOW));
		*((unsigned short *) &tmpaddr[4]) =
			be16_to_cpu(readl(fep->hwp + FEC_ADDR_HIGH) >> 16);
		iap = &tmpaddr[0];
	}
    // 随机生成
	/*
	 * 4) generate random mac 
	 */
	if (!is_valid_ether_addr(iap)) 
		random_ether_addr(iap);

	memcpy(ndev->dev_addr, iap, ETH_ALEN);

	/* Adjust MAC if using macaddr */
	if (iap == macaddr)
		 ndev->dev_addr[ETH_ALEN-1] = macaddr[ETH_ALEN-1] + fep->pdev->id;
}
```

Tony Liu

2016-9-22, Shenzhen