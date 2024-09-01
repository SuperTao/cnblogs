分析imx6 framebuffer设备和驱动的注册过程。

　　　　　　　　　　　　　　　　　　　　　　　　　　Tony Liu, 2016-8-31, Shenzhen

相关文件：

　　arch/arm/mach-mx6/board-mx6q_sabresd.c

　　kernel/video/mxc/mxc_ipuv3_fb.c

　　mm/memblock.c                                                                                 |

　　drivers/video/fbmem.c                                                             |

#### 设备注册

```
MACHINE_START(MX6Q_SABRESD, "Freescale i.MX 6Quad/DualLite/Solo Sabre-SD Board")
    /* Maintainer: Freescale Semiconductor, Inc. */
    .boot_params = MX6_PHYS_OFFSET + 0x100,
    .fixup = fixup_mxc_board,                    --------------+        //读取uboot bootargs中关于framebufer的参数
    .map_io = mx6_map_io,                                      |
    .init_irq = mx6_init_irq,                                  |
    .init_machine = mx6_sabresd_board_init,      ---------------------------------------------+     //设备初始化
    .timer = &mx6_sabresd_timer,                               |                              |
    .reserve = mx6q_sabresd_reserve,             -----------------------------------+         |     //framebuffer内存保留
MACHINE_END                                                    |                    |         |
                                                               |                    |         |
//读取bootargs中的参数                                         V                    |         |
static void __init fixup_mxc_board(struct machine_desc *desc, struct tag *tags,     |         |
                   char **cmdline, struct meminfo *mi)                              |         |
{                                                                                   |         |
    char *str;                                                                      |         |
    struct tag *t;                                                                  |         |
    int i = 0;                                                                      |         |
    struct ipuv3_fb_platform_data *pdata_fb = sabresd_fb_data;                      |         |
                                                                                    |         |
    for_each_tag(t, tags) {                                                         |         |
        if (t->hdr.tag == ATAG_CMDLINE) {                                           |         |
            str = t->u.cmdline.cmdline;                                             |         |
            str = strstr(str, "fbmem=");                                            |         |
            if (str != NULL) {                                                      |         |
                str += 6;                                                           |         |
                pdata_fb[i++].res_size[0] = memparse(str, &str);                    |         |
                while (*str == ',' &&                                               |         |
                    i < ARRAY_SIZE(sabresd_fb_data)) {                              |         |
                    str++;                                                          |         |
                    pdata_fb[i++].res_size[0] = memparse(str, &str);                |         |
                }                                                                   |         |
            }                                                                       |         |
            /* ION reserved memory */                                               |         |
            str = t->u.cmdline.cmdline;                                             |         |
            str = strstr(str, "ionmem=");                                           |         |
            if (str != NULL) {                                                      |         |
                str += 7;                                                           |         |
                imx_ion_data.heaps[0].size = memparse(str, &str);                   |         |
            }                                                                       |         |
            /* Primary framebuffer base address */                                  |         |
            str = t->u.cmdline.cmdline;                                             |         |
            str = strstr(str, "fb0base=");                                          |         |
            if (str != NULL) {                                                      |         |
                str += 8;                                                           |         |
                pdata_fb[0].res_base[0] =                                           |         |
                        simple_strtol(str, &str, 16);                               |         |
            }                                                                       |         |
            /* GPU reserved memory */                                               |         |
            str = t->u.cmdline.cmdline;                                             |         |
            str = strstr(str, "gpumem=");                                           |         |
            if (str != NULL) {                                                      |         |
                str += 7;                                                           |         |
                imx6q_gpu_pdata.reserved_mem_size = memparse(str, &str);            |         |
            }                                                                       |         |
            break;                                                                  |         |
        }                                                                           |         |
    }                                                                               |         |
}                                                                                   |         |
                                                                                    |         |
static void __init mx6q_sabresd_reserve(void)                        <------------- +         |
{                                                                                             |
    phys_addr_t phys;                                                                         |
    int i, fb0_reserved = 0, fb_array_size;                                                   |
                                                                                              |
    /*                                                                                        |
     * Reserve primary framebuffer memory if its base address                                 |
     * is set by kernel command line.                                                         |
     */                                                                                       |
    // fb0                                                                                    |
    fb_array_size = ARRAY_SIZE(sabresd_fb_data);                                              |
    if (fb_array_size > 0 && sabresd_fb_data[0].res_base[0] &&                                |
        sabresd_fb_data[0].res_size[0]) {                                                     |
        //将framebuffer的数据保留，放在memblock.reverse中
        memblock_reserve(sabresd_fb_data[0].res_base[0],                                      |
                 sabresd_fb_data[0].res_size[0]);                                             |
        //将原来的内存中memblock.memory中删除
        memblock_remove(sabresd_fb_data[0].res_base[0],                                       |
                sabresd_fb_data[0].res_size[0]);                                              |
        sabresd_fb_data[0].late_init = true;                                                  |
        ipu_data[ldb_data.ipu_id].bypass_reset = true;                                        |
        fb0_reserved = 1;                                                                     |
    }                                                                                         |
    // 对于fb_arrary剩下的元素
    for (i = fb0_reserved; i < fb_array_size; i++)                                            |
        if (sabresd_fb_data[i].res_size[0]) {                                                 |
            /* Reserve for other background buffer. */                                        |
            //申请空间，保留到memblock.reverse中
            phys = memblock_alloc(sabresd_fb_data[i].res_size[0],                             |
                        SZ_4K);                                                               |
            //将memblock.memory中一块大小的空间删除，因为已经申请，并放入memblock.reverse中
            memblock_remove(phys, sabresd_fb_data[i].res_size[0]);                            |
            sabresd_fb_data[i].res_base[0] = phys;                      --------------+       |
        }                                                                             |       |
                                                                                      |       |
#ifdef CONFIG_ANDROID_RAM_CONSOLE                                                     |       |
    //保留128K的内存到memblock.reserve，4K对齐                                        |       |
    phys = memblock_alloc_base(SZ_128K, SZ_4K, SZ_1G);                                |       |
    memblock_remove(phys, SZ_128K);                                                   |       |
    memblock_free(phys, SZ_128K);                                                     |       |
    ram_console_resource.start = phys;                                                |       |
    ram_console_resource.end   = phys + SZ_128K - 1;                                  |       |
#endif                                                                                |       |
                                                                                      |       |
#if defined(CONFIG_MXC_GPU_VIV) || defined(CONFIG_MXC_GPU_VIV_MODULE)                 |       |
    //保留  内存到memblock.reserve，4K对齐
    if (imx6q_gpu_pdata.reserved_mem_size) {                                          |       |
        phys = memblock_alloc_base(imx6q_gpu_pdata.reserved_mem_size,                 |       |
                       SZ_4K, SZ_1G);                                                 |       |
        memblock_remove(phys, imx6q_gpu_pdata.reserved_mem_size);                     |       |
        imx6q_gpu_pdata.reserved_mem_base = phys;                                     |       |
    }                                                                                 |       |
#endif                                                                                |       |
                                                                                      |       |
#if defined(CONFIG_ION)                                                               |       |
    if (imx_ion_data.heaps[0].size) {                                                 |       |
        phys = memblock_alloc(imx_ion_data.heaps[0].size, SZ_4K);                     |       |
        memblock_remove(phys, imx_ion_data.heaps[0].size);                            |       |
        imx_ion_data.heaps[0].base = phys;                                            |       |
    }                                                                                 |       |
#endif                                                                                |       |
}                                                                                     |       |
                                                                                      |       |
//#define IPU_PIX_FMT_BIT24                                                           |       |
static struct ipuv3_fb_platform_data sabresd_fb_data[] = {                   <--------+       |
    { /*fb0*/                                                                                 |
    .disp_dev = "ldb",                                                                        |
#ifdef IPU_PIX_FMT_BIT24                                                                      |
    .interface_pix_fmt = IPU_PIX_FMT_RGB24,                                                   |
    .default_bpp = 24,                                                                        |
#else                                                                                         |
    .interface_pix_fmt = IPU_PIX_FMT_RGB666,                                                  |
    .default_bpp = 16,                                                                        |
#endif                                                                                        |
    .mode_str = "LDB-XGA",                                                                    |
    .int_clk = false,                                                                         |
    .late_init = false,                                                                       |
    },                                                                                        |
    {                                                                                         |
    .disp_dev = "hdmi",                                                                       |
    .interface_pix_fmt = IPU_PIX_FMT_RGB24,                                                   |
    .mode_str = "1920x1080M@60",                                                              |
    .default_bpp = 32,                                                                        |
    .int_clk = false,                                                                         |
    .late_init = false,                                                                       |
    },                                                                                        |
    {                                                                                         |
    .disp_dev = "ldb",                                                                        |
#ifdef IPU_PIX_FMT_BIT24                                                                      |
    .interface_pix_fmt = IPU_PIX_FMT_RGB24,                                                   |
    .default_bpp = 24,                                                                        |
#else                                                                                         |
    .interface_pix_fmt = IPU_PIX_FMT_RGB666,                                                  |
    .default_bpp = 16,                                                                        |
#endif                                                                                        |
    .mode_str = "LDB-XGA",                                                                    |
    .int_clk = false,                                                                         |
    .late_init = false,                                                                       |
    },                                                                                        |
};                                                                                            |
                                                                                              |
mm/memblock.c                                                                                 |
long __init_memblock memblock_reserve(phys_addr_t base, phys_addr_t size)                     |
{                                                                                             |
    struct memblock_type *_rgn = &memblock.reserved;                                          |
                                                                                              |
    BUG_ON(0 == size);                                                                        |
                                                                                              |
    return memblock_add_region(_rgn, base, size);                                             |
}                                                                                             |
                                                                                              |
long __init_memblock memblock_remove(phys_addr_t base, phys_addr_t size)                      |
{                                                                                             |
    return __memblock_remove(&memblock.memory, base, size);                                   |
}                                                                                             |
                                                                                              |
phys_addr_t __init memblock_alloc(phys_addr_t size, phys_addr_t align)                        |
{                                                                                             |
    return memblock_alloc_base(size, align, MEMBLOCK_ALLOC_ACCESSIBLE);                       |
}                                                                                             |
                                                                                              |
static void __init mx6_sabresd_board_init(void)                              <----------------+
{
    .....
    imx6q_add_ipuv3(0, &ipu_data[0]);
    if (cpu_is_mx6q()) {
        imx6q_add_ipuv3(1, &ipu_data[1]);                                              <------+
        for (i = 0; i < 4 && i < ARRAY_SIZE(sabresd_fb_data); i++)                            |
            imx6q_add_ipuv3fb(i, &sabresd_fb_data[i]);                                        |
    } else                                                                                    |
        for (i = 0; i < 2 && i < ARRAY_SIZE(sabresd_fb_data); i++)                            |
            imx6q_add_ipuv3fb(i, &sabresd_fb_data[i]);                                        |
    ......                                                                                    |
}                                                                                             |
                                                                                              |
extern const struct imx_ipuv3_data imx6q_ipuv3_data[] __initconst;                     <------+
#define imx6q_add_ipuv3(id, pdata)    imx_add_ipuv3(id, &imx6q_ipuv3_data[id], pdata)         |
#define imx6q_add_ipuv3fb(id, pdata)    imx_add_ipuv3_fb(id, pdata)                           |
                                                                                              |
struct platform_device *__init imx_add_ipuv3_fb(                                <-------------+
        const int id,
        const struct ipuv3_fb_platform_data *pdata)
{
    if (pdata->res_size[0] > 0) {
        struct resource res[] = {
            {
                .start = pdata->res_base[0],
                .end = pdata->res_base[0] + pdata->res_size[0] - 1,
                .flags = IORESOURCE_MEM,
            }, {
                .start = 0,
                .end = 0,
                .flags = IORESOURCE_MEM,
            },
        };

        if (pdata->res_size[1] > 0) {
            res[1].start = pdata->res_base[1];
            res[1].end = pdata->res_base[1] +
                    pdata->res_size[1] - 1;
        }

        return imx_add_platform_device_dmamask("mxc_sdc_fb",
                id, res, ARRAY_SIZE(res), pdata,
                sizeof(*pdata), DMA_BIT_MASK(32));
    } else
        return imx_add_platform_device_dmamask("mxc_sdc_fb", id,
                NULL, 0, pdata, sizeof(*pdata),
                DMA_BIT_MASK(32));
}
```


#### 驱动注册

kernel/video/mxc/mxc_ipuv3_fb.c
```
int __init mxcfb_init(void)
{
    return platform_driver_register(&mxcfb_driver);       ------+
}                                                               |
                                                                |
static struct platform_driver mxcfb_driver = {             <----+
    .driver = {
           .name = MXCFB_NAME,
           },
    .probe = mxcfb_probe,             -------------------+
    .remove = mxcfb_remove,                              |
    .suspend = mxcfb_suspend,                            |
    .resume = mxcfb_resume,                              |
};                                                       |
#define MXCFB_NAME      "mxc_sdc_fb"                     |
                                                         |
                                                         |
static int mxcfb_probe(struct platform_device *pdev)  <--+
{
    struct ipuv3_fb_platform_data *plat_data = pdev->dev.platform_data;
    struct fb_info *fbi;
    struct mxcfb_info *mxcfbi;
    struct resource *res;
    struct device *disp_dev;
    char buf[32];
    int ret = 0;
                                             +--------------------------------+
    /* Initialize FB structures */           |                                |
    fbi = mxcfb_init_fbinfo(&pdev->dev, &mxcfb_ops);                          |
    if (!fbi) {                                                               |
        ret = -ENOMEM;                                                        |
        goto init_fbinfo_failed;                                              |
    }                                                                         |
                                                                              |
    ret = mxcfb_option_setup(pdev, fbi);               ----------------------------+
    if (ret)                                                                  |    |
        goto get_fb_option_failed;                                            |    |
                                                                              |    |
    mxcfbi = (struct mxcfb_info *)fbi->par;                                   |    |
    spin_lock_init(&mxcfbi->lock);                                            |    |
    mxcfbi->fbi = fbi;                                                        |    |
    mxcfbi->ipu_int_clk = plat_data->int_clk;                                 |    |
    mxcfbi->late_init = plat_data->late_init;                                 |    |
    mxcfbi->first_set_par = true;                                             |    |
    mxcfbi->panel_width_mm = plat_data->panel_width_mm;                       |    |
    mxcfbi->panel_height_mm = plat_data->panel_height_mm;                     |    |
                                                                              |    |
    ret = mxcfb_dispdrv_init(pdev, fbi);             -----------------------------------+
    if (ret < 0)                                                              |    |    |
        goto init_dispdrv_failed;                                             |    |    |
                                                                              |    |    |
    ret = ipu_test_set_usage(mxcfbi->ipu_id, mxcfbi->ipu_di);       -------------------------+
    if (ret < 0) {                                                            |    |    |    |
        dev_err(&pdev->dev, "ipu%d-di%d already in use\n",                    |    |    |    |
                mxcfbi->ipu_id, mxcfbi->ipu_di);                              |    |    |    |
        goto ipu_in_busy;                                                     |    |    |    |
    }                                                                         |    |    |    |
                                                                              |    |    |    |
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);                     |    |    |    |
    if (res && res->start && res->end) {                                      |    |    |    |
        fbi->fix.smem_len = res->end - res->start + 1;                        |    |    |    |
        fbi->fix.smem_start = res->start;                                     |    |    |    |
        fbi->screen_base = ioremap(fbi->fix.smem_start, fbi->fix.smem_len);   |    |    |    |
        /* Do not clear the fb content drawn in bootloader. */                |    |    |    |
        if (!mxcfbi->late_init)                                               |    |    |    |
            memset(fbi->screen_base, 0, fbi->fix.smem_len);                   |    |    |    |
    }                                                                         |    |    |    |
                                                                              |    |    |    |
    mxcfbi->ipu = ipu_get_soc(mxcfbi->ipu_id);                                |    |    |    |
    if (IS_ERR(mxcfbi->ipu)) {                                                |    |    |    |
        ret = -ENODEV;                                                        |    |    |    |
        goto get_ipu_failed;                                                  |    |    |    |
    }                                                                         |    |    |    |
                                                                              |    |    |    |
    /* first user uses DP with alpha feature */                               |    |    |    |
    if (!g_dp_in_use[mxcfbi->ipu_id]) {                                       |    |    |    |
        mxcfbi->ipu_ch_irq = IPU_IRQ_BG_SYNC_EOF;                             |    |    |    |
        mxcfbi->ipu_ch_nf_irq = IPU_IRQ_BG_SYNC_NFACK;                        |    |    |    |
        mxcfbi->ipu_alp_ch_irq = IPU_IRQ_BG_ALPHA_SYNC_EOF;                   |    |    |    |
        mxcfbi->ipu_vsync_pre_irq = mxcfbi->ipu_di ?                          |    |    |    |
                        IPU_IRQ_VSYNC_PRE_1 :                                 |    |    |    |
                        IPU_IRQ_VSYNC_PRE_0;                                  |    |    |    |
        mxcfbi->ipu_ch = MEM_BG_SYNC;                                         |    |    |    |
        /* Unblank the primary fb only by default */                          |    |    |    |
        if (pdev->id == 0)                                                    |    |    |    |
            mxcfbi->cur_blank = mxcfbi->next_blank = FB_BLANK_UNBLANK;        |    |    |    |
        else                                                                  |    |    |    |
            mxcfbi->cur_blank = mxcfbi->next_blank = FB_BLANK_POWERDOWN;      |    |    |    |
                                                                              |    |    |    |
        ret = mxcfb_register(fbi);                 ------------------------------------------------+
        if (ret < 0)                                                          |    |    |    |     |
            goto mxcfb_register_failed;                                       |    |    |    |     |
                                                                              |    |    |    |     |
        ipu_disp_set_global_alpha(mxcfbi->ipu, mxcfbi->ipu_ch,                |    |    |    |     |
                      true, 0x80);                                            |    |    |    |     |
        ipu_disp_set_color_key(mxcfbi->ipu, mxcfbi->ipu_ch, false, 0);        |    |    |    |     |
                                                                              |    |    |    |     |
        res = platform_get_resource(pdev, IORESOURCE_MEM, 1);                 |    |    |    |     |
        ret = mxcfb_setup_overlay(pdev, fbi, res);                            |    |    |    |     |
                                                                              |    |    |    |     |
        if (ret < 0) {                                                        |    |    |    |     |
            mxcfb_unregister(fbi);                                            |    |    |    |     |
            goto mxcfb_setupoverlay_failed;                                   |    |    |    |     |
        }                                                                     |    |    |    |     |
                                                                              |    |    |    |     |
        g_dp_in_use[mxcfbi->ipu_id] = true;                                   |    |    |    |     |
                                                                              |    |    |    |     |
        ret = device_create_file(mxcfbi->ovfbi->dev,                          |    |    |    |     |
                     &dev_attr_fsl_disp_property);                            |    |    |    |     |
        if (ret)                                                              |    |    |    |     |
            dev_err(mxcfbi->ovfbi->dev, "Error %d on creating "               |    |    |    |     |
                            "file for disp property\n",                       |    |    |    |     |
                            ret);                                             |    |    |    |     |
                                                                              |    |    |    |     |
        ret = device_create_file(mxcfbi->ovfbi->dev,                          |    |    |    |     |
                     &dev_attr_fsl_disp_dev_property);                        |    |    |    |     |
        if (ret)                                                              |    |    |    |     |
            dev_err(mxcfbi->ovfbi->dev, "Error %d on creating "               |    |    |    |     |
                            "file for disp device "                           |    |    |    |     |
                            "propety\n", ret);                                |    |    |    |     |
    } else {                                                                  |    |    |    |     |
        mxcfbi->ipu_ch_irq = IPU_IRQ_DC_SYNC_EOF;                             |    |    |    |     |
        mxcfbi->ipu_ch_nf_irq = IPU_IRQ_DC_SYNC_NFACK;                        |    |    |    |     |
        mxcfbi->ipu_alp_ch_irq = -1;                                          |    |    |    |     |
        mxcfbi->ipu_vsync_pre_irq = mxcfbi->ipu_di ?                          |    |    |    |     |
                        IPU_IRQ_VSYNC_PRE_1 :                                 |    |    |    |     |
                        IPU_IRQ_VSYNC_PRE_0;                                  |    |    |    |     |
        mxcfbi->ipu_ch = MEM_DC_SYNC;                                         |    |    |    |     |
        mxcfbi->cur_blank = mxcfbi->next_blank = FB_BLANK_POWERDOWN;          |    |    |    |     |
                                                                              |    |    |    |     |
        ret = mxcfb_register(fbi);                                            |    |    |    |     |
        if (ret < 0)                                                          |    |    |    |     |
            goto mxcfb_register_failed;                                       |    |    |    |     |
    }                                                                         |    |    |    |     |
                                                                              |    |    |    |     |
    platform_set_drvdata(pdev, fbi);                                          |    |    |    |     |
                                                                              |    |    |    |     |
    ret = device_create_file(fbi->dev, &dev_attr_fsl_disp_property);          |    |    |    |     |
    if (ret)                                                                  |    |    |    |     |
        dev_err(&pdev->dev, "Error %d on creating file for disp "             |    |    |    |     |
                    "property\n", ret);                                       |    |    |    |     |
                                                                              |    |    |    |     |
    ret = device_create_file(fbi->dev, &dev_attr_fsl_disp_dev_property);      |    |    |    |     |
    if (ret)                                                                  |    |    |    |     |
        dev_err(&pdev->dev, "Error %d on creating file for disp "             |    |    |    |     |
                    " device propety\n", ret);                                |    |    |    |     |
                                                                              |    |    |    |     |
    disp_dev = mxc_dispdrv_getdev(mxcfbi->dispdrv);                           |    |    |    |     |
    if (disp_dev) {                                                           |    |    |    |     |
        ret = sysfs_create_link(&fbi->dev->kobj,                              |    |    |    |     |
                &disp_dev->kobj, "disp_dev");                                 |    |    |    |     |
        if (ret)                                                              |    |    |    |     |
            dev_err(&pdev->dev,                                               |    |    |    |     |
                "Error %d on creating file\n", ret);                          |    |    |    |     |
    }                                                                         |    |    |    |     |
                                                                              |    |    |    |     |
    INIT_WORK(&mxcfbi->vsync_pre_work, mxcfb_vsync_pre_work);                 |    |    |    |     |
                                                                              |    |    |    |     |
    snprintf(buf, sizeof(buf), "mxcfb%d-vsync-pre", fbi->node);               |    |    |    |     |
    mxcfbi->vsync_pre_queue = create_singlethread_workqueue(buf);             |    |    |    |     |
    if (mxcfbi->vsync_pre_queue == NULL) {                                    |    |    |    |     |
        dev_err(fbi->device,                                                  |    |    |    |     |
            "Failed to alloc vsync-pre workqueue\n");                         |    |    |    |     |
        ret = -ENOMEM;                                                        |    |    |    |     |
        goto workqueue_alloc_failed;                                          |    |    |    |     |
    }                                                                         |    |    |    |     |
                                                                              |    |    |    |     |
#ifdef CONFIG_HAS_EARLYSUSPEND                                                |    |    |    |     |
    mxcfbi->fbdrv_earlysuspend.level = EARLY_SUSPEND_LEVEL_DISABLE_FB;        |    |    |    |     |
    mxcfbi->fbdrv_earlysuspend.suspend = mxcfb_early_suspend;                 |    |    |    |     |
    mxcfbi->fbdrv_earlysuspend.resume = mxcfb_later_resume;                   |    |    |    |     |
    mxcfbi->fbdrv_earlysuspend.data = pdev;                                   |    |    |    |     |
    register_early_suspend(&mxcfbi->fbdrv_earlysuspend);                      |    |    |    |     |
#endif                                                                        |    |    |    |     |
                                                                              |    |    |    |     |
#ifdef CONFIG_LOGO                                                            |    |    |    |     |
    fb_prepare_logo(fbi, 0);                                                  |    |    |    |     |
    fb_show_logo(fbi, 0);                                                     |    |    |    |     |
#endif                                                                        |    |    |    |     |
    /*                                                                        |    |    |    |     |
     * Disable HannStar touch panel CABC function,                            |    |    |    |     |
     * this function turns the panel's backlight automatically                |    |    |    |     |
     * according to the content shown on the panel which                      |    |    |    |     |
     * may cause annoying unstable backlight issue.                           |    |    |    |     |
     *                                                                        |    |    |    |     |
     * zengjf 2015-10-8 this also has down in uboot                           |    |    |    |     |
     */                                                                       |    |    |    |     |
    /*                                                                        |    |    |    |     |
    gpio_request(SABRESD_CABC_EN1, "cabc-en1");                               |    |    |    |     |
    gpio_direction_output(SABRESD_CABC_EN1, 1);                               |    |    |    |     |
    gpio_request(SABRESD_CABC_EN0, "cabc-en0");                               |    |    |    |     |
    gpio_direction_output(SABRESD_CABC_EN0, 1);                               |    |    |    |     |
    */                                                                        |    |    |    |     |
                                                                              |    |    |    |     |
    return 0;                                                                 |    |    |    |     |
                                                                              |    |    |    |     |
workqueue_alloc_failed:                                                       |    |    |    |     |
mxcfb_setupoverlay_failed:                                                    |    |    |    |     |
mxcfb_register_failed:                                                        |    |    |    |     |
get_ipu_failed:                                                               |    |    |    |     |
    ipu_clear_usage(mxcfbi->ipu_id, mxcfbi->ipu_di);                          |    |    |    |     |
ipu_in_busy:                                                                  |    |    |    |     |
init_dispdrv_failed:                                                          |    |    |    |     |
    fb_dealloc_cmap(&fbi->cmap);                                              |    |    |    |     |
    framebuffer_release(fbi);                                                 |    |    |    |     |
get_fb_option_failed:                                                         |    |    |    |     |
init_fbinfo_failed:                                                           |    |    |    |     |
    return ret;                                                               |    |    |    |     |
}                                                                             |    |    |    |     |
                                                                              |    |    |    |     |
static struct fb_ops mxcfb_ops = {                         <------------------+    |    |    |     |
    .owner = THIS_MODULE,                                                          |    |    |     |
    .fb_set_par = mxcfb_set_par,                                                   |    |    |     |
    .fb_check_var = mxcfb_check_var,                                               |    |    |     |
    .fb_setcolreg = mxcfb_setcolreg,                                               |    |    |     |
    .fb_pan_display = mxcfb_pan_display,                                           |    |    |     |
    .fb_ioctl = mxcfb_ioctl,                                                       |    |    |     |
    .fb_mmap = mxcfb_mmap,                                                         |    |    |     |
    .fb_fillrect = cfb_fillrect,                                                   |    |    |     |
    .fb_copyarea = cfb_copyarea,                                                   |    |    |     |
    .fb_imageblit = cfb_imageblit,                                                 |    |    |     |
    .fb_blank = mxcfb_blank,                                                       |    |    |     |
};                                                                                 |    |    |     |
                                                                                   |    |    |     |
                                                                                   |    |    |     |
/*                                                                                 |    |    |     |
 * Parse user specified options (`video=trident:')                                 |    |    |     |
 * example:                                                                        |    |    |     |
 *     video=mxcfb0:dev=lcd,800x480M-16@55,if=RGB565,bpp=16,noaccel                |    |    |     |
 *    video=mxcfb0:dev=lcd,800x480M-16@55,if=RGB565,fbpix=RGB565                   |    |    |     |
 */                                                                                |    |    |     |
static int mxcfb_option_setup(struct platform_device *pdev, struct fb_info *fbi) <-+    |    |     |
{                                                                                       |    |     |
    struct ipuv3_fb_platform_data *pdata = pdev->dev.platform_data;                     |    |     |
    char *options, *opt, *fb_mode_str = NULL;                                           |    |     |
    char name[] = "mxcfb0";                                                             |    |     |
    uint32_t fb_pix_fmt = 0;                                                            |    |     |
                                                                                        |    |     |
    name[5] += pdev->id;                                                                |    |     |
    if (fb_get_options(name, &options)) {                              -------------+   |    |     |
        dev_err(&pdev->dev, "Can't get fb option for %s!\n", name);                 |   |    |     |
        return -ENODEV;                                                             |   |    |     |
    }                                                                               |   |    |     |
                                                                                    |   |    |     |
    if (!options || !*options)                                                      |   |    |     |
        return 0;                                                                   |   |    |     |
                                                                                    |   |    |     |
    while ((opt = strsep(&options, ",")) != NULL) {                                 |   |    |     |
        if (!*opt)                                                                  |   |    |     |
            continue;                                                               |   |    |     |
                                                                                    |   |    |     |
        if (!strncmp(opt, "dev=", 4)) {                                             |   |    |     |
            memcpy(pdata->disp_dev, opt + 4, strlen(opt) - 4);                      |   |    |     |
            pdata->disp_dev[strlen(opt) - 4] = '\0';                                |   |    |     |
        } else if (!strncmp(opt, "if=", 3)) {                                       |   |    |     |
            if (!strncmp(opt+3, "RGB24", 5))                                        |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_RGB24;                       |   |    |     |
            else if (!strncmp(opt+3, "BGR24", 5))                                   |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_BGR24;                       |   |    |     |
            else if (!strncmp(opt+3, "GBR24", 5))                                   |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_GBR24;                       |   |    |     |
            else if (!strncmp(opt+3, "RGB565", 6))                                  |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_RGB565;                      |   |    |     |
            else if (!strncmp(opt+3, "RGB666", 6))                                  |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_RGB666;                      |   |    |     |
            else if (!strncmp(opt+3, "YUV444", 6))                                  |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_YUV444;                      |   |    |     |
            else if (!strncmp(opt+3, "LVDS666", 7))                                 |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_LVDS666;                     |   |    |     |
            else if (!strncmp(opt+3, "YUYV16", 6))                                  |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_YUYV;                        |   |    |     |
            else if (!strncmp(opt+3, "UYVY16", 6))                                  |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_UYVY;                        |   |    |     |
            else if (!strncmp(opt+3, "YVYU16", 6))                                  |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_YVYU;                        |   |    |     |
            else if (!strncmp(opt+3, "VYUY16", 6))                                  |   |    |     |
                pdata->interface_pix_fmt = IPU_PIX_FMT_VYUY;                        |   |    |     |
        } else if (!strncmp(opt, "fbpix=", 6)) {                                    |   |    |     |
            if (!strncmp(opt+6, "RGB24", 5))                                        |   |    |     |
                fb_pix_fmt = IPU_PIX_FMT_RGB24;                                     |   |    |     |
            else if (!strncmp(opt+6, "BGR24", 5))                                   |   |    |     |
                fb_pix_fmt = IPU_PIX_FMT_BGR24;                                     |   |    |     |
            else if (!strncmp(opt+6, "RGB32", 5))                                   |   |    |     |
                fb_pix_fmt = IPU_PIX_FMT_RGB32;                                     |   |    |     |
            else if (!strncmp(opt+6, "BGR32", 5))                                   |   |    |     |
                fb_pix_fmt = IPU_PIX_FMT_BGR32;                                     |   |    |     |
            else if (!strncmp(opt+6, "ABGR32", 6))                                  |   |    |     |
                fb_pix_fmt = IPU_PIX_FMT_ABGR32;                                    |   |    |     |
            else if (!strncmp(opt+6, "RGB565", 6))                                  |   |    |     |
                fb_pix_fmt = IPU_PIX_FMT_RGB565;                                    |   |    |     |
                                                                                    |   |    |     |
            if (fb_pix_fmt) {                                                       |   |    |     |
                pixfmt_to_var(fb_pix_fmt, &fbi->var);                               |   |    |     |
                pdata->default_bpp =                                                |   |    |     |
                    fbi->var.bits_per_pixel;                                        |   |    |     |
            }                                                                       |   |    |     |
        } else if (!strncmp(opt, "int_clk", 7)) {                                   |   |    |     |
            pdata->int_clk = true;                                                  |   |    |     |
            continue;                                                               |   |    |     |
        } else if (!strncmp(opt, "bpp=", 4)) {                                      |   |    |     |
            /* bpp setting cannot overwirte fbpix setting */                        |   |    |     |
            if (fb_pix_fmt)                                                         |   |    |     |
                continue;                                                           |   |    |     |
                                                                                    |   |    |     |
            pdata->default_bpp =                                                    |   |    |     |
                simple_strtoul(opt + 4, NULL, 0);                                   |   |    |     |
                                                                                    |   |    |     |
            fb_pix_fmt = bpp_to_pixfmt(pdata->default_bpp);                         |   |    |     |
            if (fb_pix_fmt)                                                         |   |    |     |
                pixfmt_to_var(fb_pix_fmt, &fbi->var);                               |   |    |     |
        } else                                                                      |   |    |     |
            fb_mode_str = opt;                                                      |   |    |     |
    }                                                                               |   |    |     |
                                                                                    |   |    |     |
    if (fb_mode_str)                                                                |   |    |     |
        pdata->mode_str = fb_mode_str;                                              |   |    |     |
                                                                                    |   |    |     |
    return 0;                                                                       |   |    |     |
}                                                                                   |   |    |     |
                                                                                    |   |    |     |
drivers/video/fbmem.c                                                               |   |    |     |
/**                                                                                 |   |    |     |
 * fb_get_options - get kernel boot parameters                                      |   |    |     |
 * @name:   framebuffer name as it would appear in                                  |   |    |     |
 *          the boot parameter line                                                 |   |    |     |
 *          (video=<name>:<options>)                                                |   |    |     |
 * @option: the option will be stored here                                          |   |    |     |
 *                                                                                  |   |    |     |
 * NOTE: Needed to maintain backwards compatibility                                 |   |    |     |
 */                                                                                 |   |    |     |
//                                                                                  |   |    |     |
int fb_get_options(char *name, char **option)                              <--------+   |    |     |
{                                                                                       |    |     |
    char *opt, *options = NULL;                                                         |    |     |
    int retval = 0;                                                                     |    |     |
    int name_len = strlen(name), i;                                                     |    |     |
                                                                                        |    |     |
    if (name_len && ofonly && strncmp(name, "offb", 4))                                 |    |     |
        retval = 1;                                                                     |    |     |
                                                                                        |    |     |
    if (name_len && !retval) {                                                          |    |     |
        for (i = 0; i < FB_MAX; i++) {                                                  |    |     |
            if (video_options[i] == NULL)                                               |    |     |
                continue;                                                               |    |     |
            if (!video_options[i][0])                                                   |    |     |
                continue;                                                               |    |     |
            opt = video_options[i];                                                     |    |     |
            if (!strncmp(name, opt, name_len) &&                                        |    |     |
                opt[name_len] == ':')                                                   |    |     |
                options = opt + name_len + 1;                                           |    |     |
        }                                                                               |    |     |
    }                                                                                   |    |     |
    if (options && !strncmp(options, "off", 3))                                         |    |     |
        retval = 1;                                                                     |    |     |
                                                                                        |    |     |
    if (option)                                                                         |    |     |
        *option = options;                                                              |    |     |
                                                                                        |    |     |
    return retval;                                                                      |    |     |
}                                                                                       |    |     |
                                                                                        |    |     |
static int mxcfb_dispdrv_init(struct platform_device *pdev,             <---------------+    |     |
        struct fb_info *fbi)                                                                 |     |
{                                                                                            |     |
    struct ipuv3_fb_platform_data *plat_data = pdev->dev.platform_data;                      |     |
    struct mxcfb_info *mxcfbi = (struct mxcfb_info *)fbi->par;                               |     |
    struct mxc_dispdrv_setting setting;                                                      |     |
    char disp_dev[32], *default_dev = "lcd";                                                 |     |
    int ret = 0;                                                                             |     |
                                                                                             |     |
    setting.if_fmt = plat_data->interface_pix_fmt;                                           |     |
    setting.dft_mode_str = plat_data->mode_str;                                              |     |
    setting.default_bpp = plat_data->default_bpp;                                            |     |
    if (!setting.default_bpp)                                                                |     |
        setting.default_bpp = 16;                                                            |     |
    setting.fbi = fbi;                                                                       |     |
    if (!strlen(plat_data->disp_dev)) {                                                      |     |
        memcpy(disp_dev, default_dev, strlen(default_dev));                                  |     |
        disp_dev[strlen(default_dev)] = '\0';                                                |     |
    } else {                                                                                 |     |
        memcpy(disp_dev, plat_data->disp_dev,                                                |     |
                strlen(plat_data->disp_dev));                                                |     |
        disp_dev[strlen(plat_data->disp_dev)] = '\0';                                        |     |
    }                                                                                        |     |
                                                                                             |     |
    dev_info(&pdev->dev, "register mxc display driver %s\n", disp_dev);                      |     |
                                                                                             |     |
    mxcfbi->dispdrv = mxc_dispdrv_gethandle(disp_dev, &setting);                             |     |
    if (IS_ERR(mxcfbi->dispdrv)) {                                                           |     |
        ret = PTR_ERR(mxcfbi->dispdrv);                                                      |     |
        dev_err(&pdev->dev, "NO mxc display driver found!\n");                               |     |
        return ret;                                                                          |     |
    } else {                                                                                 |     |
        /* fix-up  */                                                                        |     |
        mxcfbi->ipu_di_pix_fmt = setting.if_fmt;                                             |     |
        mxcfbi->default_bpp = setting.default_bpp;                                           |     |
                                                                                             |     |
        /* setting */                                                                        |     |
        mxcfbi->ipu_id = setting.dev_id;                                                     |     |
        mxcfbi->ipu_di = setting.disp_id;                                                    |     |
    }                                                                                        |     |
                                                                                             |     |
    return ret;                                                                              |     |
}                                                                                            |     |
                                                                                             |     |
static bool ipu_usage[2][2];                                                                 |     |
static int ipu_test_set_usage(int ipu, int di)                    <--------------------------+     |
{                                                                                                  |
    if (ipu_usage[ipu][di])                                                                        |
        return -EBUSY;                                                                             |
    else                                                                                           |
        ipu_usage[ipu][di] = true;                                                                 |
    return 0;                                                                                      |
}                                                                                                  |
                                                                                                   |
static int mxcfb_register(struct fb_info *fbi)                       <-----------------------------+
{
    struct mxcfb_info *mxcfbi = (struct mxcfb_info *)fbi->par;
    struct fb_videomode m;
    int ret = 0;
    char bg0_id[] = "DISP3 BG";
    char bg1_id[] = "DISP3 BG - DI1";
    char fg_id[] = "DISP3 FG";

    if (mxcfbi->ipu_di == 0) {
        bg0_id[4] += mxcfbi->ipu_id;
        strcpy(fbi->fix.id, bg0_id);
    } else if (mxcfbi->ipu_di == 1) {
        bg1_id[4] += mxcfbi->ipu_id;
        strcpy(fbi->fix.id, bg1_id);
    } else { /* Overlay */
        fg_id[4] += mxcfbi->ipu_id;
        strcpy(fbi->fix.id, fg_id);
    }

    mxcfb_check_var(&fbi->var, fbi);

    mxcfb_set_fix(fbi);

    /* Added first mode to fbi modelist. */
    if (!fbi->modelist.next || !fbi->modelist.prev)
        INIT_LIST_HEAD(&fbi->modelist);
    fb_var_to_videomode(&m, &fbi->var);
    fb_add_videomode(&m, &fbi->modelist);

    if (ipu_request_irq(mxcfbi->ipu, mxcfbi->ipu_ch_irq,
        mxcfb_irq_handler, IPU_IRQF_ONESHOT, MXCFB_NAME, fbi) != 0) {
        dev_err(fbi->device, "Error registering EOF irq handler.\n");
        ret = -EBUSY;
        goto err0;
    }
    ipu_disable_irq(mxcfbi->ipu, mxcfbi->ipu_ch_irq);
    if (ipu_request_irq(mxcfbi->ipu, mxcfbi->ipu_ch_nf_irq,
        mxcfb_nf_irq_handler, IPU_IRQF_ONESHOT, MXCFB_NAME, fbi) != 0) {
        dev_err(fbi->device, "Error registering NFACK irq handler.\n");
        ret = -EBUSY;
        goto err1;
    }
    ipu_disable_irq(mxcfbi->ipu, mxcfbi->ipu_ch_nf_irq);

    if (mxcfbi->ipu_vsync_pre_irq != -1) {
        if (ipu_request_irq(mxcfbi->ipu, mxcfbi->ipu_vsync_pre_irq,
                    mxcfb_vsync_pre_irq_handler, 0,
                    MXCFB_NAME, fbi) != 0) {
            dev_err(fbi->device, "Error registering VSYNC irq "
                         "handler.\n");
            ret = -EBUSY;
            goto err2;
        }
        ipu_disable_irq(mxcfbi->ipu, mxcfbi->ipu_vsync_pre_irq);
    }

    if (mxcfbi->ipu_alp_ch_irq != -1)
        if (ipu_request_irq(mxcfbi->ipu, mxcfbi->ipu_alp_ch_irq,
                mxcfb_alpha_irq_handler, IPU_IRQF_ONESHOT,
                    MXCFB_NAME, fbi) != 0) {
            dev_err(fbi->device, "Error registering alpha irq "
                    "handler.\n");
            ret = -EBUSY;
            goto err3;
        }

    if (!mxcfbi->late_init) {
        fbi->var.activate |= FB_ACTIVATE_FORCE;
        console_lock();
        fbi->flags |= FBINFO_MISC_USEREVENT;
        ret = fb_set_var(fbi, &fbi->var);
        fbi->flags &= ~FBINFO_MISC_USEREVENT;
        console_unlock();
        if (ret < 0) {
            dev_err(fbi->device, "Error fb_set_var ret:%d\n", ret);
            goto err4;
        }

        if (mxcfbi->next_blank == FB_BLANK_UNBLANK) {
            console_lock();
            ret = fb_blank(fbi, FB_BLANK_UNBLANK);
            console_unlock();
            if (ret < 0) {
                dev_err(fbi->device,
                    "Error fb_blank ret:%d\n", ret);
                goto err5;
            }
        }
    } else {
        /*
         * Setup the channel again though bootloader
         * has done this, then set_par() can stop the
         * channel neatly and re-initialize it .
         */
        if (mxcfbi->next_blank == FB_BLANK_UNBLANK) {
            console_lock();
            _setup_disp_channel1(fbi);
            ipu_enable_channel(mxcfbi->ipu, mxcfbi->ipu_ch);
            console_unlock();
        }
    }

    ret = register_framebuffer(fbi);                             -----------------+
    if (ret < 0)                                                                  |
        goto err6;                                                                |
                                                                                  |
    return ret;                                                                   |
err6:                                                                             |
    if (mxcfbi->next_blank == FB_BLANK_UNBLANK) {                                 |
        console_lock();                                                           |
        if (!mxcfbi->late_init)                                                   |
            fb_blank(fbi, FB_BLANK_POWERDOWN);                                    |
        else {                                                                    |
            ipu_disable_channel(mxcfbi->ipu, mxcfbi->ipu_ch,                      |
                        true);                                                    |
            ipu_uninit_channel(mxcfbi->ipu, mxcfbi->ipu_ch);                      |
        }                                                                         |
        console_unlock();                                                         |
    }                                                                             |
err5:                                                                             |
err4:                                                                             |
    if (mxcfbi->ipu_alp_ch_irq != -1)                                             |
        ipu_free_irq(mxcfbi->ipu, mxcfbi->ipu_alp_ch_irq, fbi);                   |
err3:                                                                             |
    if (mxcfbi->ipu_vsync_pre_irq != -1)                                          |
        ipu_free_irq(mxcfbi->ipu, mxcfbi->ipu_vsync_pre_irq, fbi);                |
err2:                                                                             |
    ipu_free_irq(mxcfbi->ipu, mxcfbi->ipu_ch_nf_irq, fbi);                        |
err1:                                                                             |
    ipu_free_irq(mxcfbi->ipu, mxcfbi->ipu_ch_irq, fbi);                           |
err0:                                                                             |
    return ret;                                                                   |
}                                                                                 |
                                                                                  |
drivers/video/fbmem.c                                                             |
register_framebuffer(struct fb_info *fb_info)                       <-------------+
{
    int ret;

    mutex_lock(&registration_lock);
    ret = do_register_framebuffer(fb_info);                         -----+
    mutex_unlock(&registration_lock);                                    |
                                                                         |
    return ret;                                                          |
}                                                                        |
                                                                         |
static int do_register_framebuffer(struct fb_info *fb_info)         <----+
{
    int i;
    struct fb_event event;
    struct fb_videomode mode;

    if (fb_check_foreignness(fb_info))
        return -ENOSYS;

    do_remove_conflicting_framebuffers(fb_info->apertures, fb_info->fix.id,
                     fb_is_primary_device(fb_info));

    if (num_registered_fb == FB_MAX)
        return -ENXIO;

    num_registered_fb++;
    for (i = 0 ; i < FB_MAX; i++)
        if (!registered_fb[i])
            break;
    fb_info->node = i;
    atomic_set(&fb_info->count, 1);
    mutex_init(&fb_info->lock);
    mutex_init(&fb_info->mm_lock);

    fb_info->dev = device_create(fb_class, fb_info->device,
                     MKDEV(FB_MAJOR, i), NULL, "fb%d", i);                  --------------------------->> // 设备节点创建  /dev/fb0
    if (IS_ERR(fb_info->dev)) {
        /* Not fatal */
        printk(KERN_WARNING "Unable to create device for framebuffer %d; errno = %ld\n", i, PTR_ERR(fb_info->dev));
        fb_info->dev = NULL;
    } else
        fb_init_device(fb_info);

    if (fb_info->pixmap.addr == NULL) {
        fb_info->pixmap.addr = kmalloc(FBPIXMAPSIZE, GFP_KERNEL);
        if (fb_info->pixmap.addr) {
            fb_info->pixmap.size = FBPIXMAPSIZE;
            fb_info->pixmap.buf_align = 1;
            fb_info->pixmap.scan_align = 1;
            fb_info->pixmap.access_align = 32;
            fb_info->pixmap.flags = FB_PIXMAP_DEFAULT;
        }
    }
    fb_info->pixmap.offset = 0;

    if (!fb_info->pixmap.blit_x)
        fb_info->pixmap.blit_x = ~(u32)0;

    if (!fb_info->pixmap.blit_y)
        fb_info->pixmap.blit_y = ~(u32)0;

    if (!fb_info->modelist.prev || !fb_info->modelist.next)
        INIT_LIST_HEAD(&fb_info->modelist);

    fb_var_to_videomode(&mode, &fb_info->var);
    fb_add_videomode(&mode, &fb_info->modelist);
    registered_fb[i] = fb_info;

    event.info = fb_info;
    if (!lock_fb_info(fb_info))
        return -ENODEV;
    fb_notifier_call_chain(FB_EVENT_FB_REGISTERED, &event);
    unlock_fb_info(fb_info);
    return 0;
}
```