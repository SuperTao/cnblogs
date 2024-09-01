跟踪uboot代码，了解imx6 splash image的生成过程。

涉及文件：

　　./cpu/arm_cortexa8/start.S

　　./board/freescale/mx6q_sabresd/mx6q_sabresd.c 

　　./board/freescale/commom/fsl_bmp_reversed_600x400.c

```
/* 汇编调用C语言 */
./cpu/arm_cortexa8/start.S:

ldr    pc, _start_armboot    @ jump to C code         ------+
                                                            |
lib_arm/board.c                                       <-----+
start_armboot(void)
{
    ......
    /* 一系列初始化 */
    for (init_fnc_ptr = init_sequence; *init_fnc_ptr; ++init_fnc_ptr) {   ---+
        if ((*init_fnc_ptr)() != 0) {                                        |
            hang ();                                                         |
        }                                                                    |
    }                                                                        |
    ......                                                                   |
                                                                             |
#if defined CONFIG_SPLASH_SCREEN && defined CONFIG_VIDEO_MX5                 |
    setup_splash_image();                                --------------------|-+
#endif                                                                       | |
    ......                                                                   | |
}                                                                            | |
                                                                             | |
init_fnc_t *init_sequence[] = {                                     <--------+ |
#if defined(CONFIG_ARCH_CPU_INIT)                                              |
    arch_cpu_init,        /* basic arch cpu dependent setup */                 |
#endif                                                                         |
    board_init,        /* basic board dependent setup */     ------+           |
#if defined(CONFIG_USE_IRQ)                                        |           |
    interrupt_init,        /* set up exceptions */                 |           |
#endif                                                             |           |
    timer_init,        /* initialize timer */                      |           |
    env_init,        /* initialize environment */                  |           |
    init_baudrate,        /* initialze baudrate settings */        |           |
    serial_init,        /* serial communications setup */          |           |
    console_init_f,        /* stage 1 init of console */           |           |
    display_banner,        /* say that we are here */              |           |
#if defined(CONFIG_DISPLAY_CPUINFO)                                |           |
    print_cpuinfo,        /* display cpu info (and speed) */       |           |
#endif                                                             |           |
#if defined(CONFIG_DISPLAY_BOARDINFO)                              |           |
    checkboard,        /* display board info */                    |           |
#endif                                                             |           |
#if defined(CONFIG_HARD_I2C) || defined(CONFIG_SOFT_I2C)           |           |
    init_func_i2c,                                                 |           |
#endif                                                             |           |
    dram_init,        /* configure available RAM banks */          |           |
#if defined(CONFIG_CMD_PCI) || defined (CONFIG_PCI)                |           |
    arm_pci_init,                                                  |           |
#endif                                                             |           |
    display_dram_config,                                           |           |
    NULL,                                                          |           |
};                                                                 |           |
                                                                   |           |
/* imx6初始化文件 */                                               |           |
board/freescale/mx6q_sabresd/mx6q_sabresd.c                        |           |
int board_init(void)                                         <-----+           |
{                                                                              |
/* need set Power Supply Glitch to 0x41736166                                  |
*and need clear Power supply Glitch Detect bit                                 |
* when POR or reboot or power on Otherwise system                              |
*could not be power off anymore*/                                              |
    u32 reg;                                                                   |
    writel(0x41736166, SNVS_BASE_ADDR + 0x64);/*set LPPGDR*/                   |
    udelay(10);                                                                |
    reg = readl(SNVS_BASE_ADDR + 0x4c);                                        |
    reg |= (1 << 3);                                                           |
    writel(reg, SNVS_BASE_ADDR + 0x4c);/*clear LPSR*/                          |
                                                                               |
    mxc_iomux_v3_init((void *)IOMUXC_BASE_ADDR);                               |
    setup_boot_device();                                                       |
    fsl_set_system_rev();                                                      |
                                                                               |
    /* board id for linux */                                                   |
    gd->bd->bi_arch_number = MACH_TYPE_MX6Q_SABRESD;                           |
                                                                               |
    /* address of boot parameters */                                           |
    gd->bd->bi_boot_params = PHYS_SDRAM_1 + 0x100;                             |
                                                                               |
    setup_uart();                                                              |
#ifdef CONFIG_DWC_AHSATA                                                       |
    if (cpu_is_mx6q())                                                         |
        setup_sata();                                                          |
#endif                                                                         |
                                                                               |
#ifdef CONFIG_VIDEO_MX5                                                        |
    /* Enable lvds power */                                                    |
//    setup_lvds_poweron();                                                    |
                                                                               |
    panel_info_init();                            -------------------+         |
                                                                     |         |
    gd->fb_base = CONFIG_FB_BASE;                                    |         |
#ifdef CONFIG_ARCH_MMU                                               |         |
    gd->fb_base = ioremap_nocache(iomem_to_phys(gd->fb_base), 0);    |         |
#endif                                                               |         |
#endif                                                               |         |
                                                                     |         |
#ifdef CONFIG_NAND_GPMI                                              |         |
//    setup_gpmi_nand();                                             |         |
#endif                                                               |         |
                                                                     |         |
#if defined(CONFIG_ENET_RMII) && !defined(CONFIG_DWC_AHSATA)         |         |
    setup_fec();                                                     |         |
#endif                                                               |         |
                                                                     |         |
#ifdef CONFIG_MXC_EPDC                                               |         |
    setup_epdc();                                     -------------------+     |
#endif                                                               |   |     |
    return 0;                                                        |   |     |
}                                                                    |   |     |
                                                                     |   |     |
#ifdef CONFIG_VIDEO_MX5                                              |   |     |
void panel_info_init(void)                                   <-------+   |     |
{                                                                        |     |
    panel_info.vl_bpix = LCD_BPP;        //屏幕颜色深度                  |     |
    panel_info.vl_col = lvds_xga.xres;    //屏幕尺寸                     |     |
    panel_info.vl_row = lvds_xga.yres;    //屏幕尺寸                     |     |
    panel_info.cmap = colormap;                                          |     |
}                                                                        |     |
#endif                                                                   |     |
                                                                         |     |
static void setup_epdc(void)                     <-----------------------+     |
{                                                                              |
    unsigned int reg;                                                          |
                                                                               |
    /*** epdc Maxim PMIC settings ***/                                         |
                                                                               |
    /* EPDC PWRSTAT - GPIO2[21] for PWR_GOOD status */                         |
    mxc_iomux_v3_setup_pad(MX6DL_PAD_EIM_A17__GPIO_2_21);                      |
                                                                               |
    /* EPDC VCOM0 - GPIO3[17] for VCOM control */                              |
    mxc_iomux_v3_setup_pad(MX6DL_PAD_EIM_D17__GPIO_3_17);                      |
                                                                               |
    /* UART4 TXD - GPIO3[20] for EPD PMIC WAKEUP */                            |
    mxc_iomux_v3_setup_pad(MX6DL_PAD_EIM_D20__GPIO_3_20);                      |
                                                                               |
    /* EIM_A18 - GPIO2[20] for EPD PWR CTL0 */                                 |
    mxc_iomux_v3_setup_pad(MX6DL_PAD_EIM_A18__GPIO_2_20);                      |
                                                                               |
    /*** Set pixel clock rates for EPDC ***/                                   |
                                                                               |
    /* EPDC AXI clk (IPU2_CLK) from PFD_400M, set to 396/2 = 198MHz */         |
    reg = readl(CCM_BASE_ADDR + CLKCTL_CSCDR3);                                |
    reg &= ~0x7C000;                                                           |
    reg |= (1 << 16) | (1 << 14);                                              |
    writel(reg, CCM_BASE_ADDR + CLKCTL_CSCDR3);                                |
                                                                               |
    /* EPDC AXI clk enable */                                                  |
    reg = readl(CCM_BASE_ADDR + CLKCTL_CCGR3);                                 |
    reg |= 0x00C0;                                                             |
    writel(reg, CCM_BASE_ADDR + CLKCTL_CCGR3);                                 |
                                                                               |
    /* EPDC PIX clk (IPU2_DI1_CLK) from PLL5, set to 650/4/6 = ~27MHz */       |
    reg = readl(CCM_BASE_ADDR + CLKCTL_CSCDR2);                                |
    reg &= ~0x3FE00;                                                           |
    reg |= (2 << 15) | (5 << 12);                                              |
    writel(reg, CCM_BASE_ADDR + CLKCTL_CSCDR2);                                |
                                                                               |
    /* PLL5 enable (defaults to 650) */                                        |
    reg = readl(ANATOP_BASE_ADDR + ANATOP_PLL_VIDEO);                          |
    reg &= ~((1 << 16) | (1 << 12));                                           |
    reg |= (1 << 13);                                                          |
    writel(reg, ANATOP_BASE_ADDR + ANATOP_PLL_VIDEO);                          |
                                                                               |
    /* EPDC PIX clk enable */                                                  |
    reg = readl(CCM_BASE_ADDR + CLKCTL_CCGR3);                                 |
    reg |= 0x0C00;                                                             |
    writel(reg, CCM_BASE_ADDR + CLKCTL_CCGR3);                                 |
                                                                               |
    panel_info.epdc_data.working_buf_addr = CONFIG_WORKING_BUF_ADDR;           |
    panel_info.epdc_data.waveform_buf_addr = CONFIG_WAVEFORM_BUF_ADDR;         |
                                                                               |
    panel_info.epdc_data.wv_modes.mode_init = 0;                               |
    panel_info.epdc_data.wv_modes.mode_du = 1;                                 |
    panel_info.epdc_data.wv_modes.mode_gc4 = 3;                                |
    panel_info.epdc_data.wv_modes.mode_gc8 = 2;                                |
    panel_info.epdc_data.wv_modes.mode_gc16 = 2;                               |
    panel_info.epdc_data.wv_modes.mode_gc32 = 2;                               |
                                                                               |
    panel_info.epdc_data.epdc_timings = panel_timings;                         |
                                                                               |
    setup_epdc_power();                                                        |
                                                                               |
    /* Assign fb_base */                                                       |
    gd->fb_base = CONFIG_FB_BASE;                                              |
}                                                                              |
                                                                               |
#ifdef CONFIG_SPLASH_SCREEN                                                    |
void setup_splash_image(void)                        <-------------------------+
{
    char *s;
    ulong addr;

    s = getenv("splashimage");

    if (s != NULL) {
        addr = simple_strtoul(s, NULL, 16);

#if defined(CONFIG_ARCH_MMU)
        addr = ioremap_nocache(iomem_to_phys(addr),
                fsl_bmp_reversed_600x400_size);
#endif
        /* 显示uboot图片 */
        memcpy((char *)addr, (char *)fsl_bmp_reversed_600x400,        ----+
                fsl_bmp_reversed_600x400_size);                           |
    }                                                                     |
}                                                                         |
#endif                                                                    |
/* bmp文件生成的源文件 */                                                 |
board/freescale/commom/fsl_bmp_reversed_600x400.c                         |
const unsigned char fsl_bmp_reversed_600x400[] = {                  <-----+
    ......
}
```

##### Author

Tony Liu

2016-8-9, Shenzhen