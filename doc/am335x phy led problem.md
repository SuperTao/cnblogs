问题描述
---
　　连接网线的情况下，每次进行软件"reboot"，网口的LINK LED能够正常的熄灭，而ACTIVE LED却是亮的。
reboot重启之后，LINK的灯正常变亮，而ACTIVE的灯却灭了.在有数据传输的时，能够正常的闪烁。
将网线拔了，LINK灯灭，ACTIVE灯亮。
    如果一直插上网线，每次reboot之后，ACTIVE的灯的没数据通信时，一次是亮的，下一次reboot又灭了，这样交替出现。
                  
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　Tony Liu, 2016-6-8, Shenzhen

参考链接
---
　　[Bone A6 (and others) PHY LEDs after reboot](https://groups.google.com/forum/#!topic/beagleboard/PFXV9Nrsf5U)

　　[AM335x: Cannot set PRM_RSTTIME register under Linux](http://e2e.ti.com/support/arm/sitara_arm/f/791/p/339413/1185385#pi316653filter=answers&pi316653scroll=false)

　　[Not so warm restart of AM3358 via software?](http://e2e.ti.com/support/arm/sitara_arm/f/791/t/347189#pi316653filter=all&pi316653scroll=false)

　　[DM8168复位问题疑问](http://www.deyisupport.com/question_answer/dsp_arm/davinci_digital_media_processors/f/39/p/17456/58516.aspx#58516)

　　[[U-Boot] am335x_evm: Fix Ethernet LED issue on BeagleBone](http://patchwork.ozlabs.org/patch/197551/)

　　[Not asserting RESETIN_OUT on beaglebone](https://e2e.ti.com/support/arm/sitara_arm/f/791/t/247483)

　　[AM335X Watchdog Reset](https://e2e.ti.com/support/arm/sitara_arm/f/791/p/299818/1047664#1047664)

　　[beagleboard.org](http://beagleboard.org/Community/Forums)

　　[M3352 resetin_out](https://e2e.ti.com/support/arm/sitara_arm/f/791/p/335939/1172185#1172185)

　　[AM335x Warm Reset Output Falling time](https://e2e.ti.com/support/arm/sitara_arm/f/791/t/449388)

问题分析
---
　　初步分析phy芯片没有复位。测量复位引脚。
　　![](http://images2015.cnblogs.com/blog/745188/201606/745188-20160608144048996-89811413.jpg)


　　由于LAN8710的nRST引脚需要保持低电平时间最小为100us才能够复位。
而连接的AM335x的nRESTIN_OUT引脚输出低电平的时间为700ns左右。
经过SN74LVC1G07DBV开漏输出缓冲器之后，时间也才1us,远没有达到复位的要求，导致复位异常。

解决方法
---
　　在datasheet中有提及可以是指寄存器来延长nRESTIN_OUT低电平的时间。
Technical Reference Manual的8.1.7.4 Global Warm Reset有详细介绍。
PRM_RSTTIME.RSTTIME1设置即可
8.1.13.5.2 PRM_RSTTIME Register
![](http://images2015.cnblogs.com/blog/745188/201606/745188-20160608144127121-1321378508.png)



　　在uboot中设置，进行验证

```
OK335X# md 0x44e00f04 4
44e00f04: 00001fff 00000003 78000017 00000003    ...........x....
OK335X# mw 0x44e00f04 0xffff
OK335X# md 0x44e00f04 4
44e00f04: 00001fff 00000003 78000017 00000003    ...........x....
OK335X# reset
```

　　这样最大的信号已经延长到12us左右，但是远没有达到100us，但是ACTIVE LED正常。 
![](http://images2015.cnblogs.com/blog/745188/201606/745188-20160608144140980-872466638.jpg)



　　可是这样的设置并没有kernel中生效。最后定位到,reboot命令，调用的am335x的 omap_prcm_arch_reset函数 
reboot操作启动设置PRM_RSTCTRL Register的RST_GLOBAL_COLD_SW,将其置1,这样一来直接设置的寄存器值都覆盖为默认值，设置就没有效果。
更改为设置RST_GLOBAL_WARM_SW即可，只会讲一部分的寄存器设置为默认值，我设置的PRM_RSTTIME的值并没有因为reboot命令而设置为默认值。

arch/arm/mach-omap2/prcm.c
```
void (*arch_reset)(char, const char *) = omap_prcm_arch_reset;

static void omap_prcm_arch_reset(char mode, const char *cmd)
{
    s16 prcm_offs = 0;
    unsigned int val;
	if (cpu_is_omap24xx()) {
		omap2xxx_clk_prepare_for_reboot();

		prcm_offs = WKUP_MOD;
	} else if (cpu_is_am33xx()) {
		prcm_offs = AM33XX_PRM_DEVICE_MOD;
                /* add by Tony, 2016-6-7 */
                //将PRM_RSTTIME.RSTTIME1寄存器的值设置成最大0xff，这样一来复位时就会延长低电平波形的长度，延长至12us左右。
                omap2_prm_set_mod_reg_bits(0xff,                       
                         prcm_offs, AM33XX_PRM_RSTTIME_OFFSET);

		//omap2_prm_set_mod_reg_bits(OMAP4430_RST_GLOBAL_COLD_SW_MASK,
		//			prcm_offs, AM33XX_PRM_RSTCTRL_OFFSET);
        //更改
		omap2_prm_set_mod_reg_bits(OMAP4430_RST_GLOBAL_WARM_SW_MASK,
					prcm_offs, AM33XX_PRM_RSTCTRL_OFFSET);
	} else if (cpu_is_omap34xx()) {
		prcm_offs = OMAP3430_GR_MOD;
		omap3_ctrl_write_boot_mode((cmd ? (u8)*cmd : 0));
	} else if (cpu_is_omap44xx()) {
		omap4_prminst_global_warm_sw_reset(); /* never returns */
	} else {
		WARN_ON(1);
	}
    ......
}
```
当然这不是长远之计，正确的做法是:
1. 采用GPIO引脚来控制phy的reset，只要reboot时保证对应的gpio为低电平。
2. 将nRST直接上拉，不进行复位。
3. 采用硬件上的更改,想详情请留意参考链接第一条。