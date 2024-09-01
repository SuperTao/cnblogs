本文记录imx6 uboot中关于lcd初始化的过程。

uboot中相关的文件：

　　cpu/arm_cortexa8/start.S

　　lib_arm/board.c 

　　board/freescale/mx6q_sabresd/mx6q_sabresd.c 

　　common/stdio.c 

　　common/lcd.c 

　　drivers/video/ipu_common.c

```
/* 汇编调用C语言 */
./cpu/arm_cortexa8/start.S:

ldr    pc, _start_armboot    @ jump to C code         ------+
                                                            |
lib_arm/board.c                                       <-----+
void start_armboot (void)
{
    init_fnc_t **init_fnc_ptr;
    char *s;
#if defined(CONFIG_VFD) || defined(CONFIG_LCD)
    unsigned long addr;
#endif

    /* Pointer is writable since we allocated a register for it */
    gd = (gd_t*)(_armboot_start - CONFIG_SYS_MALLOC_LEN - sizeof(gd_t));
    /* compiler optimization barrier needed for GCC >= 3.4 */
    __asm__ __volatile__("": : :"memory");

    memset ((void*)gd, 0, sizeof (gd_t));
    gd->bd = (bd_t*)((char*)gd - sizeof(bd_t));
    memset (gd->bd, 0, sizeof (bd_t));

    gd->flags |= GD_FLG_RELOC;

    monitor_flash_len = _bss_start - _armboot_start;

    for (init_fnc_ptr = init_sequence; *init_fnc_ptr; ++init_fnc_ptr) {    -------+
        if ((*init_fnc_ptr)() != 0) {                                             |
            hang ();                                                              |
        }                                                                         |
    }                                                                             |
                                                                                  |
    /* armboot_start is defined in the board-specific linker script */            |
    mem_malloc_init (_armboot_start - CONFIG_SYS_MALLOC_LEN);                     |
                                                                                  |
                                                                                  |
    stdio_init ();    /* get the devices list going. */     ------------------+   |
                                                                              |   |
    jumptable_init ();                                                        |   |
                                                                              |   |
    ......                                                                    |   |
                                                                              |   |
    for (;;) {                                                                |   |
        main_loop ();                                                         |   |
    }                                                                         |   |
                                                                              |   |
    /* NOTREACHED - no way out of command loop except booting */              |   |
}                                                                             |   |
                                                                              |   |
init_fnc_t *init_sequence[] = {                                       <-------|---+
#if defined(CONFIG_ARCH_CPU_INIT)                                             |
    arch_cpu_init,        /* basic arch cpu dependent setup */                |
#endif                                                                        |
    board_init,        /* basic board dependent setup */     -------------+   |
#if defined(CONFIG_USE_IRQ)                                               |   |
    interrupt_init,        /* set up exceptions */                        |   |
#endif                                                                    |   |
    timer_init,        /* initialize timer */                             |   |
    env_init,        /* initialize environment */                         |   |
    init_baudrate,        /* initialze baudrate settings */               |   |
    serial_init,        /* serial communications setup */                 |   |
    console_init_f,        /* stage 1 init of console */                  |   |
    display_banner,        /* say that we are here */                     |   |
#if defined(CONFIG_DISPLAY_CPUINFO)                                       |   |
    print_cpuinfo,        /* display cpu info (and speed) */              |   |
#endif                                                                    |   |
#if defined(CONFIG_DISPLAY_BOARDINFO)                                     |   |
    checkboard,        /* display board info */                           |   |
#endif                                                                    |   |
#if defined(CONFIG_HARD_I2C) || defined(CONFIG_SOFT_I2C)                  |   |
    init_func_i2c,                                                        |   |
#endif                                                                    |   |
    dram_init,        /* configure available RAM banks */                 |   |
#if defined(CONFIG_CMD_PCI) || defined (CONFIG_PCI)                       |   |
    arm_pci_init,                                                         |   |
#endif                                                                    |   |
    display_dram_config,                                                  |   |
    NULL,                                                                 |   |
};                                                                        |   |
                                                                          |   |
board/freescale/mx6q_sabresd/mx6q_sabresd.c                               |   |
int board_init(void)                                   <------------------+   |
{                                                                             |
/* need set Power Supply Glitch to 0x41736166                                 |
*and need clear Power supply Glitch Detect bit                                |
* when POR or reboot or power on Otherwise system                             |
*could not be power off anymore*/                                             |
    u32 reg;                                                                  |
    writel(0x41736166, SNVS_BASE_ADDR + 0x64);/*set LPPGDR*/                  |
    udelay(10);                                                               |
    reg = readl(SNVS_BASE_ADDR + 0x4c);                                       |
    reg |= (1 << 3);                                                          |
    writel(reg, SNVS_BASE_ADDR + 0x4c);/*clear LPSR*/                         |
                                                                              |
    mxc_iomux_v3_init((void *)IOMUXC_BASE_ADDR);                              |
    setup_boot_device();                                                      |
    fsl_set_system_rev();                                                     |
                                                                              |
    /* board id for linux */                                                  |
    gd->bd->bi_arch_number = MACH_TYPE_MX6Q_SABRESD;                          |
                                                                              |
    /* address of boot parameters */                                          |
    gd->bd->bi_boot_params = PHYS_SDRAM_1 + 0x100;                            |
                                                                              |
    setup_uart();                                                             |
#ifdef CONFIG_DWC_AHSATA                                                      |
    if (cpu_is_mx6q())                                                        |
        setup_sata();                                                         |
#endif                                                                        |
                                                                              |
#ifdef CONFIG_VIDEO_MX5                                                       |
    /* Enable lvds power */                                                   |
#ifdef UBOOT_SHOW_LOGO                                                        |
    setup_lvds_poweron();                                                     |
#endif                                                                        |
    panel_info_init();                                                        |
                                                                              |
    gd->fb_base = CONFIG_FB_BASE;                                             |
#ifdef CONFIG_ARCH_MMU                                                        |
    gd->fb_base = ioremap_nocache(iomem_to_phys(gd->fb_base), 0);             |
#endif                                                                        |
#endif                                                                        |
                                                                              |
#ifdef CONFIG_NAND_GPMI                                                       |
//    setup_gpmi_nand();                                                      |
#endif                                                                        |
                                                                              |
#if defined(CONFIG_ENET_RMII) && !defined(CONFIG_DWC_AHSATA)                  |
    setup_fec();                                                              |
#endif                                                                        |
                                                                              |
#ifdef CONFIG_MXC_EPDC                                                        |
    setup_epdc();                                                             |
#endif                                                                        |
    return 0;                                                                 |
}                                                                             |
                                                                              |
common/stdio.c                                                                |
int stdio_init (void)                           <-----------------------------+
{
#ifndef CONFIG_ARM    /* already relocated for current ARM implementation */
    ulong relocation_offset = gd->reloc_off;
    int i;

    /* relocate device name pointers */
    for (i = 0; i < (sizeof (stdio_names) / sizeof (char *)); ++i) {
        stdio_names[i] = (char *) (((ulong) stdio_names[i]) +
                        relocation_offset);
    }
#endif

    /* Initialize the list */
    INIT_LIST_HEAD(&(devs.list));    //设备链表头

#ifdef CONFIG_ARM_DCC_MULTI
    drv_arm_dcc_init ();
#endif
#if defined(CONFIG_HARD_I2C) || defined(CONFIG_SOFT_I2C)
    i2c_init (CONFIG_SYS_I2C_SPEED, CONFIG_SYS_I2C_SLAVE);
#endif
#ifdef CONFIG_LCD
    drv_lcd_init ();                    ----------------------+
#endif                                                        |
#if defined(CONFIG_VIDEO) || defined(CONFIG_CFB_CONSOLE)      |
    drv_video_init ();                                        |
#endif                                                        |
#ifdef CONFIG_KEYBOARD                                        |
    drv_keyboard_init ();                                     |
#endif                                                        |
#ifdef CONFIG_LOGBUFFER                                       |
    drv_logbuff_init ();                                      |
#endif                                                        |
    drv_system_init ();                                       |
#ifdef CONFIG_SERIAL_MULTI                                    |
    serial_stdio_init ();                                     |
#endif                                                        |
#ifdef CONFIG_USB_TTY                                         |
    drv_usbtty_init ();                                       |
#endif                                                        |
#ifdef CONFIG_NETCONSOLE                                      |
    drv_nc_init ();                                           |
#endif                                                        |
#ifdef CONFIG_JTAG_CONSOLE                                    |
    drv_jtag_console_init ();                                 |
#endif                                                        |
                                                              |
    return (0);                                               |
}                                                             |
                                                              |
common/lcd.c                                                  |
int drv_lcd_init (void)               <-----------------------+
{
    struct stdio_dev lcddev;
    int rc;
    //framebuffer基地址
    lcd_base = (void *)(gd->fb_base);
    //计算多少行
    lcd_line_length = (panel_info.vl_col * NBITS (panel_info.vl_bpix)) / 8;

    lcd_init (lcd_base);        /* LCD initialization */    -----------------------------+
                                                                                         |
    /* Device initialization */                                                          |
    memset (&lcddev, 0, sizeof (lcddev));                                                |
                                                                                         |
    strcpy (lcddev.name, "lcd");                                                         |
    lcddev.ext   = 0;            /* No extensions */                                     |
    lcddev.flags = DEV_FLAGS_OUTPUT;    /* Output only */                                |
    lcddev.putc  = lcd_putc;        /* 'putc' function */                                |
    lcddev.puts  = lcd_puts;        /* 'puts' function */                                |
                                                                                         |
    rc = stdio_register (&lcddev);    //将lcd设备添加到uboot的设备链表中                 |
                                                                                         |
    return (rc == 0) ? 1 : rc;                                                           |
}                                                                                        |
                                                                                         |
#define    NBITS(type)    (sizeof(type) * 8)                                             |
                                                                                         |
board/freescale/mx6q_sabresd/mx6q_sabresd.c                                              |
vidinfo_t panel_info = {                                                                 |
    .vl_refresh = 85,                                                                    |
    .vl_col = 800,                                                                       |
    .vl_row = 600,                                                                       |
    .vl_pixclock = 26666667,                                                             |
    .vl_left_margin = 8,                                                                 |
    .vl_right_margin = 100,                                                              |
    .vl_upper_margin = 4,                                                                |
    .vl_lower_margin = 8,                                                                |
    .vl_hsync = 4,                                                                       |
    .vl_vsync = 1,                                                                       |
    .vl_sync = 0,                                                                        |
    .vl_mode = 0,                                                                        |
    .vl_flag = 0,                                                                        |
    .vl_bpix = 3,                                                                        |
    cmap:0,                                                                              |
};                                                                                       |
typedef struct vidinfo {                                                                 |
    ushort    vl_col;        /* Number of columns (i.e. 640) */                          |
    ushort    vl_row;        /* Number of rows (i.e. 480) */                             |
    ushort    vl_width;    /* Width of display area in millimeters */                    |
    ushort    vl_height;    /* Height of display area in millimeters */                  |
                                                                                         |
    /* LCD configuration register */                                                     |
    u_char    vl_clkp;    /* Clock polarity */                                           |
    u_char    vl_oep;        /* Output Enable polarity */                                |
    u_char    vl_hsp;        /* Horizontal Sync polarity */                              |
    u_char    vl_vsp;        /* Vertical Sync polarity */                                |
    u_char    vl_dp;        /* Data polarity */                                          |
    u_char    vl_bpix;    /* Bits per pixel, 0 = 1, 1 = 2, 2 = 4, 3 = 8, 4 = 16 */       |
    u_char    vl_lbw;        /* LCD Bus width, 0 = 4, 1 = 8 */                           |
    u_char    vl_splt;    /* Split display, 0 = single-scan, 1 = dual-scan */            |
    u_char    vl_clor;    /* Color, 0 = mono, 1 = color */                               |
    u_char    vl_tft;        /* 0 = passive, 1 = TFT */                                  |
                                                                                         |
    /* Horizontal control register. Timing from data sheet */                            |
    ushort    vl_hpw;        /* Horz sync pulse width */                                 |
    u_char    vl_blw;        /* Wait before of line */                                   |
    u_char    vl_elw;        /* Wait end of line */                                      |
                                                                                         |
    /* Vertical control register. */                                                     |
    u_char    vl_vpw;        /* Vertical sync pulse width */                             |
    u_char    vl_bfw;        /* Wait before of frame */                                  |
    u_char    vl_efw;        /* Wait end of frame */                                     |
                                                                                         |
    /* PXA LCD controller params */                                                      |
    struct    pxafb_info pxa;                                                            |
} vidinfo_t;                                                                             |
                                                                                         |
static int lcd_init (void *lcdbase)                                 <--------------------+
{
    /* Initialize the lcd controller */
    debug ("[LCD] Initializing LCD frambuffer at %p\n", lcdbase);

    lcd_ctrl_init (lcdbase);
    lcd_is_enabled = 1;
    lcd_clear (NULL, 1, 1, NULL);    /* dummy args */           -------------------+
    lcd_enable ();                                           ---------------------------+
                                                                                   |    |
    /* Initialize the console */                                                   |    |
    console_col = 0;                                                               |    |
#ifdef CONFIG_LCD_INFO_BELOW_LOGO                                                  |    |
    console_row = 7 + BMP_LOGO_HEIGHT / VIDEO_FONT_HEIGHT;                         |    |
#else                                                                              |    |
    console_row = 1;    /* leave 1 blank line below logo */                        |    |
#endif                                                                             |    |
                                                                                   |    |
    return 0;                                                                      |    |
}                                                                                  |    |
                                                                                   |    |
void lcd_ctrl_init(void *lcdbase)                                                  |    |
{                                                                                  |    |
    //计算要使用的framebuffer大小 col*row*bpp/8 字节                               |    |
    u32 mem_len = panel_info.vl_col *                                              |    |
        panel_info.vl_row *                                                        |    |
        NBITS(panel_info.vl_bpix) / 8;                                             |    |
                                                                                   |    |
    /*                                                                             |    |
     * We rely on lcdbase being a physical address, i.e., either MMU off,          |    |
     * or 1-to-1 mapping. Might want to add some virt2phys here.                   |    |
     */                                                                            |    |
    if (!lcdbase)                                                                  |    |
        return;                                                                    |    |
    //将framebuffer清0                                                             |    |
    memset(lcdbase, 0, mem_len);                                                   |    |
}                                                                                  |    |
                                                                                   |    |
common/lcd.c                                                                       |    |
static int lcd_clear (cmd_tbl_t * cmdtp, int flag, int argc, char *argv[])   <-----+    |
{                                                                                       |
#if LCD_BPP == LCD_MONOCHROME                                                           |
    /* Setting the palette */                                                           |
    lcd_initcolregs();                                                                  |
                                                                                        |
#elif LCD_BPP == LCD_COLOR8                                                             |
    /* Setting the palette */                                                           |
    lcd_setcolreg  (CONSOLE_COLOR_BLACK,       0,    0,    0);                          |
    lcd_setcolreg  (CONSOLE_COLOR_RED,    0xFF,    0,    0);                            |
    lcd_setcolreg  (CONSOLE_COLOR_GREEN,       0, 0xFF,    0);                          |
    lcd_setcolreg  (CONSOLE_COLOR_YELLOW,    0xFF, 0xFF,    0);                         |
    lcd_setcolreg  (CONSOLE_COLOR_BLUE,        0,    0, 0xFF);                          |
    lcd_setcolreg  (CONSOLE_COLOR_MAGENTA,    0xFF,    0, 0xFF);                        |
    lcd_setcolreg  (CONSOLE_COLOR_CYAN,       0, 0xFF, 0xFF);                           |
    lcd_setcolreg  (CONSOLE_COLOR_GREY,    0xAA, 0xAA, 0xAA);                           |
    lcd_setcolreg  (CONSOLE_COLOR_WHITE,    0xFF, 0xFF, 0xFF);                          |
#endif                                                                                  |
                                                                                        |
#ifndef CONFIG_SYS_WHITE_ON_BLACK                                                       |
    lcd_setfgcolor (CONSOLE_COLOR_BLACK);                                               |
    lcd_setbgcolor (CONSOLE_COLOR_WHITE);                                               |
#else                                                                                   |
    lcd_setfgcolor (CONSOLE_COLOR_WHITE);                                               |
    lcd_setbgcolor (CONSOLE_COLOR_BLACK);                                               |
#endif    /* CONFIG_SYS_WHITE_ON_BLACK */                                               |
                                                                                        |
#ifdef    LCD_TEST_PATTERN                                                              |
    test_pattern();                                                                     |
#else                                                                                   |
    /* set framebuffer to background color */                                           |
    memset ((char *)lcd_base,                                                           |
        COLOR_MASK(lcd_getbgcolor()),                                                   |
        lcd_line_length*panel_info.vl_row);                                             |
#endif                                                                                  |
    /* Paint the logo and retrieve LCD base address */                                  |
    debug ("[LCD] Drawing the logo...\n");                                              |
    lcd_console_address = lcd_logo ();         --------+                                |
                                                       |                                |
    console_col = 0;                                   |                                |
    console_row = 0;                                   |                                |
                                                       |                                |
    return (0);                                        |                                |
}                                                      |                                |
//显示logo                                             |                                |
static void *lcd_logo (void)                    <------+                                |
{                                                                                       |
#ifdef CONFIG_SPLASH_SCREEN                                                             |
    char *s;                                                                            |
    ulong addr;                                                                         |
    static int do_splash = 1;                                                           |
                                                                                        |
    // get the "splashimage=0x30000000\0"                                               |
    if (do_splash && (s = getenv("splashimage")) != NULL) {                             |
        int x = 0, y = 0;                                                               |
        do_splash = 0;                                                                  |
                                                                                        |
        // get the image address, 地址转为16进制                                        |
        addr = simple_strtoul (s, NULL, 16);                                            |
                                                                                        |
#ifdef CONFIG_SPLASH_SCREEN_ALIGN                                                       |
        // get the "splashpos=m,m\0"                                                    |
        if ((s = getenv ("splashpos")) != NULL) {                                       |
            // the first position x and y, default was center                           |
            if (s[0] == 'm')                                                            |
                x = BMP_ALIGN_CENTER;                                                   |
            else                                                                        |
                x = simple_strtol (s, NULL, 0);                                         |
                                                                                        |
            if ((s = strchr (s + 1, ',')) != NULL) {                                    |
                if (s[1] == 'm')                                                        |
                    y = BMP_ALIGN_CENTER;                                               |
                else                                                                    |
                    y = simple_strtol (s + 1, NULL, 0);                                 |
            }                                                                           |
        }                                                                               |
#endif /* CONFIG_SPLASH_SCREEN_ALIGN */                                                 |
                                                                                        |
#ifdef CONFIG_VIDEO_BMP_GZIP                                                            |
        // get the image struct                                                         |
        bmp_image_t *bmp = (bmp_image_t *)addr;                                         |
        unsigned long len;                                                              |
                                                                                        |
        if (!((bmp->header.signature[0]=='B') &&                                        |
              (bmp->header.signature[1]=='M'))) {                                       |
            addr = (ulong)gunzip_bmp(addr, &len);                                       |
        }                                                                               |
#endif                                                                                  |
                                                                                        |
        //显示logo,根据x,y坐标                                                          |
        if (lcd_display_bitmap (addr, x, y) == 0) {    ----------------------+          |
            return ((void *)lcd_base);                                       |          |
        }                                                                    |          |
    }                                                                        |          |
#endif /* CONFIG_SPLASH_SCREEN */                                            |          |
                                                                             |          |
#ifdef CONFIG_LCD_LOGO                                                       |          |
    bitmap_plot (0, 0);                                                      |          |
#endif /* CONFIG_LCD_LOGO */                                                 |          |
                                                                             |          |
#ifdef CONFIG_LCD_INFO                                                       |          |
    console_col = LCD_INFO_X / VIDEO_FONT_WIDTH;                             |          |
    console_row = LCD_INFO_Y / VIDEO_FONT_HEIGHT;                            |          |
    lcd_show_board_info();                                                   |          |
#endif /* CONFIG_LCD_INFO */                                                 |          |
                                                                             |          |
#if defined(CONFIG_LCD_LOGO) && !defined(CONFIG_LCD_INFO_BELOW_LOGO)         |          |
    return ((void *)((ulong)lcd_base + BMP_LOGO_HEIGHT * lcd_line_length));  |          |
#else                                                                        |          |
    return ((void *)lcd_base);                                               |          |
#endif /* CONFIG_LCD_LOGO && !CONFIG_LCD_INFO_BELOW_LOGO */                  |          |
}                                                                            |          |
                                                                             |          |
int lcd_display_bitmap(ulong bmp_image, int x, int y)        <---------------+          |
{                                                                                       |
#if !defined(CONFIG_MCC200)                                                             |
    ushort *cmap = NULL;                                                                |
#endif                                                                                  |
    ushort *cmap_base = NULL;                                                           |
    ushort i, j;                                                                        |
    uchar *fb;                                                                          |
    bmp_image_t *bmp=(bmp_image_t *)bmp_image;                                          |
    uchar *bmap;                                                                        |
    ushort padded_line;                                                                 |
    unsigned long width, height, byte_width;                                            |
    unsigned long pwidth = panel_info.vl_col;                                           |
    unsigned colors, bpix, bmp_bpix;                                                    |
    unsigned long compression;                                                          |
#if defined(CONFIG_PXA250)                                                              |
    struct pxafb_info *fbi = &panel_info.pxa;                                           |
#elif defined(CONFIG_MPC823)                                                            |
    volatile immap_t *immr = (immap_t *) CONFIG_SYS_IMMR;                               |
    volatile cpm8xx_t *cp = &(immr->im_cpm);                                            |
#endif                                                                                  |
                                                                                        |
    if (!((bmp->header.signature[0]=='B') &&                                            |
        (bmp->header.signature[1]=='M'))) {                                             |
        printf ("Error: no valid bmp image at %lx\n", bmp_image);                       |
        return 1;                                                                       |
    }                                                                                   |
                                                                                        |
    width = le32_to_cpu (bmp->header.width);                                            |
    height = le32_to_cpu (bmp->header.height);                                          |
    bmp_bpix = le16_to_cpu(bmp->header.bit_count);                                      |
    colors = 1 << bmp_bpix;                                                             |
    compression = le32_to_cpu (bmp->header.compression);                                |
                                                                                        |
    bpix = NBITS(panel_info.vl_bpix);                                                   |
                                                                                        |
    if ((bpix != 1) && (bpix != 8) && (bpix != 16) && (bpix != 24)) {                   |
        printf ("Error: %d bit/pixel mode, but BMP has %d bit/pixel\n",                 |
            bpix, bmp_bpix);                                                            |
        return 1;                                                                       |
    }                                                                                   |
                                                                                        |
#if defined(CONFIG_BMP_24BPP)                                                           |
    /* We support displaying 24bpp BMPs on 16bpp LCDs */                                |
    if (bpix != bmp_bpix && (bmp_bpix != 24 || bpix != 16) &&                           |
        (bmp_bpix != 8 || bpix != 16)) {                                                |
#else                                                                                   |
    /* We support displaying 8bpp BMPs on 16bpp LCDs */                                 |
    if (bpix != bmp_bpix && (bmp_bpix != 8 || bpix != 16)) {                            |
#endif                                                                                  |
        printf ("Error: %d bit/pixel mode, but BMP has %d bit/pixel\n",                 |
            bpix,                                                                       |
            le16_to_cpu(bmp->header.bit_count));                                        |
        return 1;                                                                       |
    }                                                                                   |
                                                                                        |
    debug ("Display-bmp: %d x %d  with %d colors\n",                                    |
        (int)width, (int)height, (int)colors);                                          |
                                                                                        |
#if !defined(CONFIG_MCC200)                                                             |
    /* MCC200 LCD doesn't need CMAP, supports 1bpp b&w only */                          |
    if (bmp_bpix == 8) {                                                                |
#if defined(CONFIG_PXA250)                                                              |
        cmap = (ushort *)fbi->palette;                                                  |
#elif defined(CONFIG_MPC823)                                                            |
        cmap = (ushort *)&(cp->lcd_cmap[255*sizeof(ushort)]);                           |
#elif !defined(CONFIG_ATMEL_LCD)                                                        |
        cmap = panel_info.cmap;                                                         |
#endif                                                                                  |
        cmap_base = cmap;                                                               |
                                                                                        |
        /* Set color map */                                                             |
        for (i=0; i<colors; ++i) {                                                      |
            bmp_color_table_entry_t cte = bmp->color_table[i];                          |
#if !defined(CONFIG_ATMEL_LCD)                                                          |
            ushort colreg =                                                             |
                ( ((cte.red)   << 8) & 0xf800) |                                        |
                ( ((cte.green) << 3) & 0x07e0) |                                        |
                ( ((cte.blue)  >> 3) & 0x001f) ;                                        |
#ifdef CONFIG_SYS_INVERT_COLORS                                                         |
            *cmap = 0xffff - colreg;                                                    |
#else                                                                                   |
            *cmap = colreg;                                                             |
#endif                                                                                  |
#if defined(CONFIG_MPC823)                                                              |
            cmap--;                                                                     |
#else                                                                                   |
            cmap++;                                                                     |
#endif                                                                                  |
#else /* CONFIG_ATMEL_LCD */                                                            |
            lcd_setcolreg(i, cte.red, cte.green, cte.blue);                             |
#endif                                                                                  |
        }                                                                               |
    }                                                                                   |
#endif                                                                                  |
                                                                                        |
#if defined(CONFIG_MCC200)                                                              |
    if (bpix==1)                                                                        |
    {                                                                                   |
        width = ((width + 7) & ~7) >> 3;                                                |
        x     = ((x + 7) & ~7) >> 3;                                                    |
        pwidth= ((pwidth + 7) & ~7) >> 3;                                               |
    }                                                                                   |
#endif                                                                                  |
    padded_line = (width&0x3) ? ((width&~0x3)+4) : (width);                             |
#ifdef CONFIG_SPLASH_SCREEN_ALIGN                                                       |
    if (x == BMP_ALIGN_CENTER)                                                          |
        x = max(0, (pwidth - width) / 2);                                               |
    else if (x < 0)                                                                     |
        x = max(0, pwidth - width + x + 1);                                             |
                                                                                        |
    if (y == BMP_ALIGN_CENTER)                                                          |
        y = max(0, (panel_info.vl_row - height) / 2);                                   |
    else if (y < 0)                                                                     |
        y = max(0, panel_info.vl_row - height + y + 1);                                 |
#endif /* CONFIG_SPLASH_SCREEN_ALIGN */                                                 |
                                                                                        |
    if ((x + width)>pwidth)                                                             |
        width = pwidth - x;                                                             |
    if ((y + height)>panel_info.vl_row)                                                 |
        height = panel_info.vl_row - y;                                                 |
                                                                                        |
    bmap = (uchar *)bmp + le32_to_cpu (bmp->header.data_offset);                        |
    fb   = (uchar *) (lcd_base +                                                        |
        (y + height - 1) * lcd_line_length + x * bpix / 8);                             |
                                                                                        |
    switch (bmp_bpix) {                                                                 |
    case 1: /* pass through */                                                          |
    case 8:                                                                             |
        if (bpix != 16)                                                                 |
            byte_width = width;                                                         |
        else                                                                            |
            byte_width = width * 2;                                                     |
                                                                                        |
        for (i = 0; i < height; ++i) {                                                  |
            WATCHDOG_RESET();                                                           |
            for (j = 0; j < width; j++) {                                               |
                if (bpix != 16) {                                                       |
#if defined(CONFIG_PXA250) || defined(CONFIG_ATMEL_LCD)                                 |
                    *(fb++) = *(bmap++);                                                |
#elif defined(CONFIG_MPC823) || defined(CONFIG_MCC200)                                  |
                    *(fb++) = 255 - *(bmap++);                                          |
#endif                                                                                  |
                } else {                                                                |
                    *(uint16_t *)fb = cmap_base[*(bmap++)];                             |
                    fb += sizeof(uint16_t) / sizeof(*fb);                               |
                }                                                                       |
            }                                                                           |
            bmap += (width - padded_line);                                              |
            fb   -= (byte_width + lcd_line_length);                                     |
        }                                                                               |
        break;                                                                          |
                                                                                        |
#if defined(CONFIG_BMP_16BPP)                                                           |
    case 16:                                                                            |
        for (i = 0; i < height; ++i) {                                                  |
            WATCHDOG_RESET();                                                           |
            for (j = 0; j < width; j++) {                                               |
#if defined(CONFIG_ATMEL_LCD_BGR555)                                                    |
                *(fb++) = ((bmap[0] & 0x1f) << 2) |                                     |
                    (bmap[1] & 0x03);                                                   |
                *(fb++) = (bmap[0] & 0xe0) |                                            |
                    ((bmap[1] & 0x7c) >> 2);                                            |
                bmap += 2;                                                              |
#else                                                                                   |
                *(fb++) = *(bmap++);                                                    |
                *(fb++) = *(bmap++);                                                    |
#endif                                                                                  |
            }                                                                           |
            bmap += (padded_line - width) * 2;                                          |
            fb   -= (width * 2 + lcd_line_length);                                      |
        }                                                                               |
        break;                                                                          |
#endif /* CONFIG_BMP_16BPP */                                                           |
#if defined(CONFIG_BMP_24BPP)                                                           |
    case 24:                                                                            |
        if (bpix != 16) {                                                               |
            printf("Error: %d bit/pixel mode,"                                          |
                "but BMP has %d bit/pixel\n",                                           |
                bpix, bmp_bpix);                                                        |
            break;                                                                      |
        }                                                                               |
        for (i = 0; i < height; ++i) {                                                  |
            WATCHDOG_RESET();                                                           |
            for (j = 0; j < width; j++) {                                               |
                *(uint16_t *)fb = ((*(bmap + 2) << 8) & 0xf800)                         |
                        | ((*(bmap + 1) << 3) & 0x07e0)                                 |
                        | ((*(bmap) >> 3) & 0x001f);                                    |
                bmap += 3;                                                              |
                fb += sizeof(uint16_t) / sizeof(*fb);                                   |
            }                                                                           |
            bmap += (width - padded_line);                                              |
            fb   -= ((2*width) + lcd_line_length);                                      |
        }                                                                               |
        break;                                                                          |
#endif /* CONFIG_BMP_24BPP */                                                           |
    default:                                                                            |
        break;                                                                          |
    };                                                                                  |
                                                                                        |
    return (0);                                                                         |
}                                                                                       |
#endif                                                                                  |
                                                                                        |
board/freescale/mx6q_sabresd/mx6q_sabresd.c                                             |
#ifndef CONFIG_MXC_EPDC                                                                 |
#ifdef CONFIG_LCD                                                                       |
void lcd_enable(void)                                    <----------------------------- +
{
    char *s;
    int ret;
    unsigned int reg;
    s = getenv("lvds_num");
    di = simple_strtol(s, NULL, 10);

    /*
    * hw_rev 2: IPUV3DEX
    * hw_rev 3: IPUV3M
    * hw_rev 4: IPUV3H
    */
    g_ipu_hw_rev = IPUV3_HW_REV_IPUV3H;

#if defined CONFIG_MX6Q
    /* PWM backlight */
    mxc_iomux_v3_setup_pad(MX6Q_PAD_SD1_DAT3__PWM1_PWMO);
    /* LVDS panel CABC_EN0 */
    mxc_iomux_v3_setup_pad(MX6Q_PAD_NANDF_CS2__GPIO_6_15);
    /* LVDS panel CABC_EN1 */
    mxc_iomux_v3_setup_pad(MX6Q_PAD_NANDF_CS3__GPIO_6_16);
#elif defined CONFIG_MX6DL
    /* PWM backlight */
    mxc_iomux_v3_setup_pad(MX6DL_PAD_SD1_DAT3__PWM1_PWMO);
    /* LVDS panel CABC_EN0 */
    mxc_iomux_v3_setup_pad(MX6DL_PAD_NANDF_CS2__GPIO_6_15);
    /* LVDS panel CABC_EN1 */
    mxc_iomux_v3_setup_pad(MX6DL_PAD_NANDF_CS3__GPIO_6_16);
#endif
    imx_pwm_config(pwm0, 25000, 50000);
    imx_pwm_enable(pwm0);

    mxc_iomux_v3_setup_pad(MX6DL_PAD_SD1_DAT2__GPIO_1_19);
    reg = readl(GPIO1_BASE_ADDR + GPIO_GDIR);
    reg |= (1 << 19);
    writel(reg, GPIO1_BASE_ADDR + GPIO_GDIR);

    reg = readl(GPIO1_BASE_ADDR + GPIO_DR);
#ifdef UBOOT_SHOW_LOGO
    reg |= (1 << 19);
#else
    reg &= ~(1 << 19);
#endif
    writel(reg, GPIO1_BASE_ADDR + GPIO_DR);

    /* Disable ipu1_clk/ipu1_di_clk_x/ldb_dix_clk. */
    reg = readl(CCM_BASE_ADDR + CLKCTL_CCGR3);
    reg &= ~0xC033;
    writel(reg, CCM_BASE_ADDR + CLKCTL_CCGR3);

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
    // 20C_403C
    reg = readl(CCM_BASE_ADDR + CLKCTL_CSCDR3);
    /* source */
    //ipu1_hsp_clk_sel
    reg |= (0x3 << 9);
    /* divider */
    reg &= ~(0x7 << 11);
    reg |= (0x1 << 11);
    //540M/2 = 270M
    writel(reg, CCM_BASE_ADDR + CLKCTL_CSCDR3);

    /*
     * ipu1_pixel_clk_x clock tree:
     * osc_clk(24M)->pll2_528_bus_main_clk(528M)->
     * pll2_pfd_352M(452.57M)->ldb_dix_clk(64.65M)->
     * ipu1_di_clk_x(64.65M)->ipu1_pixel_clk_x(64.65M)
     */
    /* pll2_528_bus_main_clk */
    writel(0x1, ANATOP_BASE_ADDR + 0x34);

    /* pll2_pfd_352M */
    writel(0x1 << 7, ANATOP_BASE_ADDR + 0x104);
    /* divider */
    writel(0x3F, ANATOP_BASE_ADDR + 0x108);
    //This field controls the fractional divide value. The resulting frequency shall be 480*18/PFD0_FRAC where
    writel(0x15, ANATOP_BASE_ADDR + 0x104);

    /* ldb_dix_clk */
    /* source */
    reg = readl(CCM_BASE_ADDR + CLKCTL_CS2CDR);
    reg &= ~(0x3F << 9);
    reg |= (0x9 << 9);
    writel(reg, CCM_BASE_ADDR + CLKCTL_CS2CDR);
    /* divider */
    reg = readl(CCM_BASE_ADDR + CLKCTL_CSCMR2);
    reg |= (0x3 << 10);
    writel(reg, CCM_BASE_ADDR + CLKCTL_CSCMR2);

    /* pll2_pfd_352M */
    /* enable after ldb_dix_clk source is set */
    writel(0x1 << 7, ANATOP_BASE_ADDR + 0x108);

    /* ipu1_di_clk_x */
    /* source */
    reg = readl(CCM_BASE_ADDR + CLKCTL_CHSCCDR);
    reg &= ~0xE07;
    reg |= 0x803;
    writel(reg, CCM_BASE_ADDR + CLKCTL_CHSCCDR);
#endif    /* CONFIG_MX6DL */

    /* Enable ipu1/ipu1_dix/ldb_dix clocks. */
    if (di == 1) {
        reg = readl(CCM_BASE_ADDR + CLKCTL_CCGR3);
        reg |= 0xC033;
        writel(reg, CCM_BASE_ADDR + CLKCTL_CCGR3);
    } else {
        reg = readl(CCM_BASE_ADDR + CLKCTL_CCGR3);
        reg |= 0x300F;
        writel(reg, CCM_BASE_ADDR + CLKCTL_CCGR3);
    }
    ret = ipuv3_fb_init(&lvds_xga, di, IPU_PIX_FMT_RGB666, DI_PCLK_LDB, 65000000); //18 bit -----+
    //ret = ipuv3_fb_init(&lvds_xga, di, IPU_PIX_FMT_RGB24, DI_PCLK_LDB, 65000000);  //24 bit    |
                                                      |            |                             |
    if (ret)                                          |            |                             |
        puts("LCD cannot be configured \n");          +--------------------+                     |
                                                                   |       |                     |
#ifdef I2C_GPIO_EEPROM_PROCESS        //WRITE process              |       |                     |
            //write singal byte                                    |       |                     |
#if 1                                                              |       |                     |
        {                                                          |       |                     |
            Load_config_from_mmc();                                |       |                     |
        }                                                          |       |                     |
#endif                                                             |       |                     |
#endif                                                             |       |                     |
                                                                   |       |                     |
    /*                                                             |       |                     |
     * LVDS0 mux to IPU1 DI0.                                      |       |                     |
     * LVDS1 mux to IPU1 DI1.                                      |       |                     |
     */                                                            |       |                     |
#ifdef UBOOT_SHOW_LOGO                                             |       |                     |
    reg = readl(IOMUXC_BASE_ADDR + 0xC);                           |       |                     |
    reg &= ~(0x000003C0);                                          |       |                     |
    reg |= 0x00000100;                                             |       |                     |
    writel(reg, IOMUXC_BASE_ADDR + 0xC);                           |       |                     |
#endif                                                             |       |                     |
    if (di == 1)                                                   |       |                     |
    {                                                              |       |                     |
        writel(0x40C, IOMUXC_BASE_ADDR + 0x8);//18BIT              |       |                     |
        //writel(0x48C, IOMUXC_BASE_ADDR + 0x8);//24BIT            |       |                     |
    }                                                              |       |                     |
    else                                                           |       |                     |
    {                                                              |       |                     |
        writel(0x201, IOMUXC_BASE_ADDR + 0x8);//18BIT              |       |                     |
        //writel(0x221, IOMUXC_BASE_ADDR + 0x8);//24BIT            |       |                     |
    }                                                              |       |                     |
    ......                                                         |       |
}                                                                  |       |                     |
#endif                                                             |       |                     |
                                                                   |       |                     |
                                                                   |       |                     |
include/ipu.h                                                      |       |                     |
typedef enum {                                                     |       |                     |
    DI_PCLK_PLL3,                                                  |       |                     |
    DI_PCLK_LDB,                               <-------------------+       |                     |
    DI_PCLK_TVE                                                            |                     |
} ipu_di_clk_parent_t;                                                     |                     |
                                                                           V                     |
#define IPU_PIX_FMT_RGB666  fourcc('R', 'G', 'B', '6')    /*< 18  RGB-6-6-6    */                |
#define IPU_PIX_FMT_RGB24   fourcc('R', 'G', 'B', '3')    /*< 24  RGB-8-8-8    */                |
                                                                                                 |
#define fourcc(a, b, c, d)\                                                                      |
    (((__u32)(a)<<0)|((__u32)(b)<<8)|((__u32)(c)<<16)|((__u32)(d)<<24))                          |
                                                                                                 |
drivers/video/mxc_ipuv3_fb.c                                                                     |
int ipuv3_fb_init(struct fb_videomode *mode, int di, int interface_pix_fmt,          <-----------+
          ipu_di_clk_parent_t di_clk_parent, int di_clk_val)
{
    int ret;

    ret = ipu_probe(di, di_clk_parent, di_clk_val);                              ----------+
    if (ret)                                                                               |
        puts("Error initializing IPU\n");                                                  |
                                                                                           |
    debug("Framebuffer at 0x%x\n", (unsigned int)lcd_base);                                |
    ret = mxcfb_probe(interface_pix_fmt, mode, di);                              --------------+
                                                                                           |   |
    return ret;                                                                            |   |
}                                                                                          |   |
                                                                                           |   |
drivers/video/ipu_common.c                                                                 |   |
int ipu_probe(int di, ipu_di_clk_parent_t di_clk_parent, int di_clk_val)          <--------+   |
{                                                                                              |
    unsigned long ipu_base;                                                                    |
                                                                                               |
#if defined(CONFIG_MXC_HSC)                                                                    |
    u32 temp;                                                                                  |
    u32 *reg_hsc_mcd = (u32 *)MIPI_HSC_BASE_ADDR;                                              |
    u32 *reg_hsc_mxt_conf = (u32 *)(MIPI_HSC_BASE_ADDR + 0x800);                               |
                                                                                               |
     __raw_writel(0xF00, reg_hsc_mcd);                                                         |
                                                                                               |
    /* CSI mode reserved*/                                                                     |
    temp = __raw_readl(reg_hsc_mxt_conf);                                                      |
     __raw_writel(temp | 0x0FF, reg_hsc_mxt_conf);                                             |
                                                                                               |
    temp = __raw_readl(reg_hsc_mxt_conf);                                                      |
    __raw_writel(temp | 0x10000, reg_hsc_mxt_conf);                                            |
#endif                                                                                         |
                                                                                               |
    ipu_base = IPU_CTRL_BASE_ADDR;                                                             |
    /* base fixup */                                                                           |
    if (g_ipu_hw_rev == IPUV3_HW_REV_IPUV3H)    /* IPUv3H */                                   |
        ipu_base += IPUV3H_REG_BASE;                                                           |
    else if (g_ipu_hw_rev == IPUV3_HW_REV_IPUV3M)    /* IPUv3M */                              |
        ipu_base += IPUV3M_REG_BASE;                                                           |
    else            /* IPUv3D, v3E, v3EX */                                                    |
        ipu_base += IPUV3DEX_REG_BASE;                                                         |
    ipu_cpmem_base = (u32 *)(ipu_base + IPU_CPMEM_REG_BASE);                                   |
    ipu_dc_tmpl_reg = (u32 *)(ipu_base + IPU_DC_TMPL_REG_BASE);                                |
                                                                                               |
    g_pixel_clk[0] = &pixel_clk[0];                                                            |
    g_pixel_clk[1] = &pixel_clk[1];                                                            |
                                                                                               |
    g_di_clk[0] = &di_clk[0];                                                                  |
    g_di_clk[1] = &di_clk[1];                                                                  |
    g_di_clk[di]->rate = di_clk_val;                                                           |
                                                                                               |
    g_ipu_clk = &ipu_clk;                                                                      |
    debug("ipu_clk = %u\n", clk_get_rate(g_ipu_clk));                                          |
                                                                                               |
    ipu_reset();                                                                               |
                                                                                               |
    if (di_clk_parent == DI_PCLK_LDB) {                                                        |
        clk_set_parent(g_pixel_clk[di], g_di_clk[di]);                                         |
    } else {                                                                                   |
        clk_set_parent(g_pixel_clk[0], g_ipu_clk);                                             |
        clk_set_parent(g_pixel_clk[1], g_ipu_clk);                                             |
    }                                                                                          |
                                                                                               |
    clk_enable(g_ipu_clk);                                                                     |
                                                                                               |
    __raw_writel(0x807FFFFF, IPU_MEM_RST);                                                     |
    while (__raw_readl(IPU_MEM_RST) & 0x80000000)                                              |
        ;                                                                                      |
                                                                                               |
    ipu_init_dc_mappings();                                                                    |
                                                                                               |
    __raw_writel(0, IPU_INT_CTRL(5));                                                          |
    __raw_writel(0, IPU_INT_CTRL(6));                                                          |
    __raw_writel(0, IPU_INT_CTRL(9));                                                          |
    __raw_writel(0, IPU_INT_CTRL(10));                                                         |
                                                                                               |
    /* DMFC Init */                                                                            |
    ipu_dmfc_init(DMFC_NORMAL, 1);                                                             |
                                                                                               |
    /* Set sync refresh channels as high priority */                                           |
    __raw_writel(0x18800000L, IDMAC_CHA_PRI(0));                                               |
                                                                                               |
    /* Set MCU_T to divide MCU access window into 2 */                                         |
    __raw_writel(0x00400000L | (IPU_MCU_T_DEFAULT << 18), IPU_DISP_GEN);                       |
                                                                                               |
    clk_disable(g_ipu_clk);                                                                    |
                                                                                               |
    return 0;                                                                                  |
}                                                                                              |
                                                                                               |
/*                                                                                             |
 * Probe routine for the framebuffer driver. It is called during the                           |
 * driver binding process.      The following functions are performed in                       |
 * this routine: Framebuffer initialization, Memory allocation and                             |
 * mapping, Framebuffer registration, IPU initialization.                                      |
 *                                                                                             |
 * @return      Appropriate error code to the kernel common code                               |
 */                                                                                            |
static int mxcfb_probe(u32 interface_pix_fmt, struct fb_videomode *mode, int di)        <------+
{
    struct fb_info *fbi;
    struct mxcfb_info *mxcfbi;
    int ret = 0;

    /*
     * Initialize FB structures
     */
    fbi = mxcfb_init_fbinfo();
    if (!fbi) {
        ret = -ENOMEM;
        goto err0;
    }
    mxcfbi = (struct mxcfb_info *)fbi->par;

    if (!g_dp_in_use) {
        mxcfbi->ipu_ch = MEM_BG_SYNC;
        mxcfbi->blank = FB_BLANK_UNBLANK;
    } else {
        mxcfbi->ipu_ch = MEM_DC_SYNC;
        mxcfbi->blank = FB_BLANK_POWERDOWN;
    }

    mxcfbi->ipu_di = di;

    ipu_disp_set_global_alpha(mxcfbi->ipu_ch, 1, 0x80);
    ipu_disp_set_color_key(mxcfbi->ipu_ch, 0, 0);
    strcpy(fbi->fix.id, "DISP3 BG");

    g_dp_in_use = 1;

    mxcfb_info[mxcfbi->ipu_di] = fbi;

    /* Need dummy values until real panel is configured */
    fbi->var.xres = 640;
    fbi->var.yres = 480;

    fbi->var.bits_per_pixel = 16;
    //fbi->var.bits_per_pixel = 24;

    mxcfbi->ipu_di_pix_fmt = interface_pix_fmt;
    fb_videomode_to_var(&fbi->var, mode);

    mxcfb_check_var(&fbi->var, fbi);

    /* Default Y virtual size is 2x panel size */
    fbi->var.yres_virtual = fbi->var.yres * 2;

    mxcfb_set_fix(fbi);

    /* alocate fb first */
    if (mxcfb_map_video_memory(fbi) < 0)
        return -ENOMEM;

    mxcfb_set_par(fbi);

    /* Setting panel_info for lcd */
    panel_info.vl_col = fbi->var.xres;
    panel_info.vl_row = fbi->var.yres;
    panel_info.vl_bpix = LCD_BPP;

    lcd_line_length = (panel_info.vl_col * NBITS(panel_info.vl_bpix)) / 8;

    debug("MXC IPUV3 configured\n"
        "XRES = %d YRES = %d BitsXpixel = %d\n",
        panel_info.vl_col,
        panel_info.vl_row,
        panel_info.vl_bpix);

    ipu_dump_registers();

    return 0;

err0:
    return ret;
}
```

##### Author

Tony Liu

2016-8-23, Shenzhen