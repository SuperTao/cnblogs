前段时间查看了uboot的时钟，kernel的也稍微了解了下，记录于此，以后再来补充完善。

```
board-mx6q_sabresd.c

MACHINE_START(MX6Q_SABRESD, "Freescale i.MX 6Quad/DualLite/Solo Sabre-SD Board")
    /* Maintainer: Freescale Semiconductor, Inc. */
    .boot_params = MX6_PHYS_OFFSET + 0x100,
    .fixup = fixup_mxc_board,
    .map_io = mx6_map_io,
    .init_irq = mx6_init_irq,
    .init_machine = mx6_sabresd_board_init,
    .timer = &mx6_sabresd_timer,             ---------+
    .reserve = mx6q_sabresd_reserve,                  |
MACHINE_END                                           |
                                                      |
static struct sys_timer mx6_sabresd_timer = {    <----+
    .init   = mx6_sabresd_timer_init,            -----+
};                                                    |
                                                      |
static void __init mx6_sabresd_timer_init(void)  <----+
{
    struct clk *uart_clk;
#ifdef CONFIG_LOCAL_TIMERS
    twd_base = ioremap(LOCAL_TWD_ADDR, SZ_256);
    BUG_ON(!twd_base);
#endif
    mx6_clocks_init(32768, 24000000, 0, 0);               -------------+
                                                                       |
    uart_clk = clk_get_sys("imx-uart.0", NULL);                        |
    early_console_setup(UART1_BASE_ADDR, uart_clk);                    |
}                                                                      |
                                                                       |
int __init mx6_clocks_init(unsigned long ckil, unsigned long osc,  <---+
    unsigned long ckih1, unsigned long ckih2)
{
    __iomem void *base;
    int i, reg;
    u32 parent_rate, rate;
    unsigned long ipg_clk_rate, max_arm_wait_clk;

    external_low_reference = ckil;    //32768
    external_high_reference = ckih1;    //0
    ckih2_reference = ckih2;        //0
    oscillator_reference = osc;        //24000000
    // 2080000 + 18000 = 2098000
    timer_base = ioremap(GPT_BASE_ADDR, SZ_4K);
    //    208000 + 48000 = 20C8000
    apll_base = ioremap(ANATOP_BASE_ADDR, SZ_4K);

    for (i = 0; i < ARRAY_SIZE(lookups); i++) {
        clkdev_add(&lookups[i]);
        clk_debug_register(lookups[i].clk);
    }
    ......

    /*
     * We don't set ipu1_di_clk[1]'s parent clock to
     * pll5_video_main_clk as bootloader may need
     * the parent to be ldb_di1_clk to support LVDS
     * panel splashimage.
     */
    clk_set_parent(&ipu1_di_clk[0], &pll5_video_main_clk);       -----------+
#ifndef CONFIG_MX6_CLK_FOR_BOOTUI_TRANS                                     |
    clk_set_parent(&ipu1_di_clk[1], &pll5_video_main_clk);                  |
#endif                                                                      |
    clk_set_parent(&ipu2_di_clk[0], &pll5_video_main_clk);                  |
    clk_set_parent(&ipu2_di_clk[1], &pll5_video_main_clk);                  |
                                                                            |
    clk_set_parent(&emi_clk, &pll2_pfd_400M);                               |
    clk_set_rate(&emi_clk, 200000000);                                      |
                                                                            |
               ...........                                                  |
}                                                                           |
                                                                            |
static struct clk ipu1_di_clk[] = {                      <------------------+
    {

     __INIT_CLK_DEBUG(ipu1_di_clk_0)
    .id = 0,
    .parent = &pll5_video_main_clk,
    .enable_reg = MXC_CCM_CCGR3,
    .enable_shift = MXC_CCM_CCGRx_CG1_OFFSET,
    .enable = _clk_enable,
    .disable = _clk_disable,
    .set_parent = _clk_ipu1_di0_set_parent,                             -------+
    .set_rate = _clk_ipu1_di0_set_rate,                                        |
    .round_rate = _clk_ipu_di_round_rate,                                      |
    .get_rate = _clk_ipu1_di0_get_rate,                                        |
    .flags = AHB_HIGH_SET_POINT | CPU_FREQ_TRIG_UPDATE,                        |
    },                                                                         |
    {                                                                          |
     __INIT_CLK_DEBUG(ipu1_di_clk_1)                                           |
    .id = 1,                                                                   |
    .parent = &pll5_video_main_clk,                                            |
    .enable_reg = MXC_CCM_CCGR3,                                               |
    .enable_shift = MXC_CCM_CCGRx_CG2_OFFSET,                                  |
    .enable = _clk_enable,                                                     |
    .disable = _clk_disable,                                                   |
    .set_parent = _clk_ipu1_di1_set_parent,                                    |
    .set_rate = _clk_ipu1_di1_set_rate,                                        |
    .round_rate = _clk_ipu_di_round_rate,                                      |
    .get_rate = _clk_ipu1_di1_get_rate,                                        |
    .flags = AHB_HIGH_SET_POINT | CPU_FREQ_TRIG_UPDATE,                        |
    },                                                                         |
};                                                                             |
                                                                               |
static int _clk_ipu1_di0_set_parent(struct clk *clk, struct clk *parent)  <----+
{
    u32 reg, mux;

    if (parent == &ldb_di0_clk)
        mux = 0x3;
    else if (parent == &ldb_di1_clk)
        mux = 0x4;
    else {
        reg = __raw_readl(MXC_CCM_CHSCCDR)
            & ~MXC_CCM_CHSCCDR_IPU1_DI0_PRE_CLK_SEL_MASK;

        mux = _get_mux6(parent, &mmdc_ch0_axi_clk[0],
                &pll3_usb_otg_main_clk, &pll5_video_main_clk,
                &pll2_pfd_352M, &pll2_pfd_400M, &pll3_pfd_540M);
        reg |= (mux << MXC_CCM_CHSCCDR_IPU1_DI0_PRE_CLK_SEL_OFFSET);

        __raw_writel(reg, MXC_CCM_CHSCCDR);

        /* Derive clock from divided pre-muxed ipu1_di0 clock.*/
        mux = 0;
    }

    reg = __raw_readl(MXC_CCM_CHSCCDR)
        & ~MXC_CCM_CHSCCDR_IPU1_DI0_CLK_SEL_MASK;
    __raw_writel(reg | (mux << MXC_CCM_CHSCCDR_IPU1_DI0_CLK_SEL_OFFSET),
        MXC_CCM_CHSCCDR);

    return 0;
}
```

Tony Liu

2016-8-30, Shenzhen