在uboot中添加logo，lvds接口的lcd显示不正常，出现波动。网上说是lvds时钟频率的问题。

使用示波器测量之后，发现频率是60M，而lcd最大频率才46.8M。

因此就需要更改uboot中lvds的时钟，本文介绍lvds的时钟配置。 

### 参考链接：

　　https://community.nxp.com/docs/DOC-172312

　　https://community.nxp.com/docs/DOC-93617

　　https://community.nxp.com/thread/306801

　　https://community.nxp.com/thread/355690

　　https://community.nxp.com/thread/378887

　　https://community.nxp.com/thread/305115

　　https://community.nxp.com/thread/395103

### 原理分析

datasheet: IMX6SDLRM

imx6 使用了2个晶振:

　　外部低频时钟:   32kHz or 32.768kHz 

　　外部高频时钟:   24MHz

![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160823155945355-2021348902.png)

![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160823155957511-51684046.png)

![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160823160008386-1111205091.png)

LVDS的时钟就是图中LDB_DI0_IPU和LDB_DI0_IPU

18.5.1.3 PLL reference clock
```
There are several PLLs in this chip.
PLL1 - ARM PLL (typical functional frequency 800 MHz)
PLL2 - System PLL (functional frequency 528 MHz)
PLL3 - USB1 PLL (functional frequency 480 MHz)
PLL4 - Audio PLL
PLL5 - Video PLL
PLL6 - ENET PLL
PLL7 - USB2 PLL (functional frequency 480 MHz)
PLL8 - MLB PLL
```

18.5.1.3.1 ARM PLL
```
This PLL synthesizes a low jitter clock from a 24 MHz reference clock. The clock output
frequency for this PLL ranges from 650 MHz to 1.3 GHz. The output frequency is
selected by a 7-bit register field CCM_ANALOG_PLL_ARM[DIV_SELECT].
PLL output frequency = Fref * DIV_SEL/2
```
CCM_ANALOG_PLL_ARM寄存器

![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160823161215698-2075883447.png)

![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160823160101855-1844923144.png)

根据寄存器计算：pll0输出范围

　　24M * 54  / 2 =  648M

　　24M * 108 / 2 =  1296M

18.5.1.3.3 System PLL

```
This PLL synthesizes a low jitter clock from the 24 MHz reference clock. The PLL has
one output clock, plus 3 PFD outputs. The System PLL supports spread spectrum
modulation for use in applications to minimize radiated emissions. The spread spectrum
PLL output clock is frequency modulated so that the energy is spread over a wider
bandwidth, thereby reducing peak radiated emissions. Due to this feature support, the
associated lock time of this PLL is longer than other PLLs in the SoC that do not support
spread spectrum modulation.
......
Although this PLL does have a DIV_SELECT register field, it is intended that this PLL
will only be run at the default frequency of 528 MHz.
```
![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160823160303230-1342413301.png)

Analog System PLL Control Register

　　24M * 20 = 480M 

　　24M * 22 = 528M

![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160823162045917-661425101.png)

18.5.1.4 Phase Fractional Dividers (PFD)
```
There are several PFD outputs from the System PLL and USB1 PLL.
Each PFD output generates a fractional multiplication of the associated PLL’s VCO
frequency. Where the output frequency is equal to Fvco*18/N, N can range from 12-35.
The PFDs allow for clock frequency changes without forcing the relock of the root PLL.
This feature is useful in support of dynamic voltage and frequency scaling (DVFS). See
CCM Analog Memory Map/Register Definition.
```
![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160823160154886-444943838.png)

![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160823160205151-929051321.png)

频率最小值：

　　480M * 18 / 35 =  246M

　　246M / 7 =  35M

### 代码更改

board/freescale/mx6q_sabresd/mx6q_sabresd.c

```
void lcd_enable(void)
{
    ......
#elif defined CONFIG_MX6DL /* CONFIG_MX6Q */
	/*
	 * IPU1 HSP clock tree:
	 * osc_clk(24M)->pll3_usb_otg_main_clk(480M)->
	 * pll3_pfd_540M(540M)->ipu1_clk(270M)
	 */
	/* pll3_usb_otg_main_clk */
	/* divider */
	writel(0x3, ANATOP_BASE_ADDR + 0x18);

	/* pll3_pfd_540M */
	/* divider */
	writel(0x3F << 8, ANATOP_BASE_ADDR + 0xF8);
	writel(0x10 << 8, ANATOP_BASE_ADDR + 0xF4);
	/* enable */
	writel(0x1 << 15, ANATOP_BASE_ADDR + 0xF8);

	/* ipu1_clk */
	reg = readl(CCM_BASE_ADDR + CLKCTL_CSCDR3);
	/* source */
	reg |= (0x3 << 9);
	/* divider */
	reg &= ~(0x7 << 11);
	reg |= (0x1 << 11);
	writel(reg, CCM_BASE_ADDR + CLKCTL_CSCDR3);

	/*
	 * ipu1_pixel_clk_x clock tree:
	 * osc_clk(24M)->pll2_528_bus_main_clk(528M)->
	 * pll2_pfd_352M(452.57M)->ldb_dix_clk(64.65M)->
	 * ipu1_di_clk_x(64.65M)->ipu1_pixel_clk_x(64.65M)
	 */
	/* pll2_528_bus_main_clk */
	/* divider */
    //Tony
//1. ----------  将pll2由528M更改为480M    ------------------
    reg = readl(ANATOP_BASE_ADDR + 0x34);  
    reg &= ~(1 << 0);                      // 24M * 20 = 480M
	writel(reg, ANATOP_BASE_ADDR + 0x34);
    ////////////
	//writel(0x1, ANATOP_BASE_ADDR + 0x34); // 24M * 22 = 528M

	/* pll2_pfd_352M */
	/* disable */
	writel(0x1 << 7, ANATOP_BASE_ADDR + 0x104);
	/* divider */
//2. ---------更改分频参数为35.  480 * 18 / 35 = 246M ----------
	writel(0x3F, ANATOP_BASE_ADDR + 0x108);
	writel(0x23, ANATOP_BASE_ADDR + 0x104);
    //原来设置 528M * 18 / 21 = 452.57M
//	writel(0x3F, ANATOP_BASE_ADDR + 0x108);
//	writel(0x15, ANATOP_BASE_ADDR + 0x104);

	/* ldb_dix_clk */
	/* source */
//3. --------- 选择时钟源, pll2_pfd0 -----------
    //ldb_di1_clk_sel Selector for ldb_di1 clock multiplexer
    //NOTE: Multiplexor should be updated when both input and output clocks are gated.
    //000 pll5 clock
    //001 derive clock from PLL2 PFD0
    //010 derive clock from PLL2 PFD2
    //011 derive clock from mmdc_ch1 clock
    //100 derive clock from pll3_sw_clk
    //101-111 Reserve
	reg = readl(CCM_BASE_ADDR + CLKCTL_CS2CDR);
	reg |= (0x9 << 9);

	writel(reg, CCM_BASE_ADDR + CLKCTL_CS2CDR);
	/* divider */
	reg = readl(CCM_BASE_ADDR + CLKCTL_CSCMR2);
	reg |= (0x3 << 10);
	writel(reg, CCM_BASE_ADDR + CLKCTL_CSCMR2);

	/* pll2_pfd_352M */
	/* enable after ldb_dix_clk source is set */
	writel(0x1 << 7, ANATOP_BASE_ADDR + 0x108);
    //ipu1 di1 root clock multiplexer : derive clock from ldb_di1_clk
    //ipu1 di0 root clock multiplexer : derive clock from ldb_di0_clk
	reg = readl(CCM_BASE_ADDR + CLKCTL_CHSCCDR);
	reg &= ~0xE07;
	reg |= 0x803;
    //ipu1 di1 root clock multiplexer : derive clock from ldb_di1_clk
    //ipu1 di0 root clock multiplexer : derive clock from ldb_di0_clk
	writel(reg, CCM_BASE_ADDR + CLKCTL_CHSCCDR);
#endif	/* CONFIG_MX6DL */
    ......
```

### Author

Tony Liu 

2016-8-23, Shenzhen