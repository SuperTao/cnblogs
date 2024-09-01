查看imx6 kernel中lvds设备和驱动的初始化过程。

相关文档：
　　arm/arm/mach-mx6/board-mx6q_sabresd.c

　　kernel/drivers/video/mxc/ldb.c

#### 设备初始化
```
arm/arm/mach-mx6/board-mx6q_sabresd.c
MACHINE_START(MX6Q_SABRESD, "Freescale i.MX 6Quad/DualLite/Solo Sabre-SD Board")
	/* Maintainer: Freescale Semiconductor, Inc. */
	.boot_params = MX6_PHYS_OFFSET + 0x100,
	.fixup = fixup_mxc_board,
	.map_io = mx6_map_io,
	.init_irq = mx6_init_irq,
	.init_machine = mx6_sabresd_board_init,    ----------+
	.timer = &mx6_sabresd_timer,                         |
	.reserve = mx6q_sabresd_reserve,                     |
MACHINE_END                                              |
                                                         |
// board-mx6q_sabresd.c                                  |
static void __init mx6_sabresd_board_init(void)      <---+
{                            ------------------------------+
    ......                  /                              |
    imx6q_add_ipuv3(0, &ipu_data[0]);          ---+        |   // ipu
                                                  |        |
    imx6q_add_ldb(&ldb_data);            ----------------------------+   //ldb
    ......                                        |        |         |
}                                                 |        |         |
                                                  V        |         |
#define imx6q_add_ipuv3(id, pdata) \                       |         |
    imx_add_ipuv3(id, &imx6q_ipuv3_data[id], pdata)        |         |
                                                           |         |
struct platform_device *__init imx_add_ipuv3(              |         |
        const int id,                                      |         |
        const struct imx_ipuv3_data *data,                 |         |
        struct imx_ipuv3_platform_data *pdata)             |         |
{                                                          |         |
    struct resource res[] = {                              |         |
        {                                                  |         |
            .start = data->iobase,                         |         |
            .end = data->iobase + data->iosize - 1,        |         |
            .flags = IORESOURCE_MEM,                       |         |
        }, {                                               |         |
            .start = data->irq_err,                        |         |
            .end = data->irq_err,                          |         |
            .flags = IORESOURCE_IRQ,                       |         |
        }, {                                               |         |
            .start = data->irq,                            |         |
            .end = data->irq,                              |         |
            .flags = IORESOURCE_IRQ,                       |         |
        },                                                 |         |
    };                                                     |         |
                                                           |         |
    pdata->init = data->init;                              |         |
    pdata->pg = data->pg;                                  |         |
                                                           |         |
    return imx_add_platform_device_dmamask("imx-ipuv3", id,|         |
            res, ARRAY_SIZE(res), pdata, sizeof(*pdata),   |         |
            DMA_BIT_MASK(32));                             |         |
}                                                          |         |
                                                           |         |
static struct imx_ipuv3_platform_data ipu_data[] = {    <--+         |
    {                                                                |
    .rev = 4,                                                        |
    .csi_clk[0] = "clko_clk",                                        |
    .bypass_reset = false,                                           |
    }, {                                                             |
    .rev = 4,                                                        |
    .csi_clk[0] = "clko_clk",                                        |
    .bypass_reset = false,                                           |
    },                                                               |
};                                                                   |
                                                                     |
#define imx6q_add_ldb(pdata) \                         <-------------+
    imx_add_ldb(&imx6q_ldb_data, pdata);                             |
                                                                     |
struct platform_device *__init imx_add_ldb(                          |
        const struct imx_ldb_data *data,                             |
        struct fsl_mxc_ldb_platform_data *pdata)                     |
{                                                                    |
    struct resource res[] = {                                        |
        {                                                            |
            .start = data->iobase,                                   |
            .end = data->iobase + data->iosize - 1,                  |
            .flags = IORESOURCE_MEM,                                 |
        },                                                           |
    };                                                               |
                                                                     |
    return imx_add_platform_device("mxc_ldb", -1,                    |
            res, ARRAY_SIZE(res), pdata, sizeof(*pdata));            |
}                                                                    |
                                                                     |
static struct fsl_mxc_ldb_platform_data ldb_data = {          <------+
    .ipu_id = 0,
    .disp_id = 1,
    .ext_ref = 1,
    .mode = LDB_SEP1,
    .sec_ipu_id = 0,
    .sec_disp_id = 0,
};
```

#### 驱动初始化

```
kernel/drivers/video/mxc/ldb.c

static int __init ldb_init(void)
{
    return platform_driver_register(&mxcldb_driver);      -----+
}                                                              |
                                                               |
static struct platform_driver mxcldb_driver = {            <---+
    .driver = {
           .name = "mxc_ldb",
           },
    .probe = ldb_probe,                              -----+
    .remove = ldb_remove,                                 |
    .suspend = ldb_suspend,                               |
    .resume = ldb_resume,                                 |
};                                                        |
                                                          |
static int ldb_probe(struct platform_device *pdev)    <---+
{
    int ret = 0;
    struct ldb_data *ldb;

    ldb = kzalloc(sizeof(struct ldb_data), GFP_KERNEL);
    if (!ldb) {
        ret = -ENOMEM;
        goto alloc_failed;
    }
                                              +------------------+
    ldb->pdev = pdev;                         |                  |
    ldb->disp_ldb = mxc_dispdrv_register(&ldb_drv);              |
    mxc_dispdrv_setdata(ldb->disp_ldb, ldb);                     |
                                                                 |
    dev_set_drvdata(&pdev->dev, ldb);                            |
                                                                 |
    /*                                                           |
     * Disable HannStar touch panel CABC function,               |
     * this function turns the panel's backlight automatically   |
     * according to the content shown on the panel which         |
     * may cause annoying unstable backlight issue.              |
     *                                                           |
     */                                                          |
                                                                 |
alloc_failed:                                                    |
    return ret;                                                  |
}                                                                |
                                                                 |
                                                                 |
static struct mxc_dispdrv_driver ldb_drv = {             <-------+
    .name     = DISPDRV_LDB,
    .init     = ldb_disp_init,           ----------------------+
    .deinit    = ldb_disp_deinit,                              |
    .setup = ldb_disp_setup,             ----------------------------------------------+
};                                                             |                       |
                                                               |                       |
static int ldb_disp_init(struct mxc_dispdrv_handle *disp,   <--+                       |
    struct mxc_dispdrv_setting *setting)                                               |
{                                                                                      |
    int ret = 0, i;                                                                    |
    struct ldb_data *ldb = mxc_dispdrv_getdata(disp);                                  |
    struct fsl_mxc_ldb_platform_data *plat_data = ldb->pdev->dev.platform_data;        |
    struct i2c_client *i2c_dev;                                                        |
    struct resource *res;                                                              |
    uint32_t base_addr;                                                                |
    uint32_t reg, setting_idx;                                                         |
    uint32_t ch_mask = 0, ch_val = 0;                                                  |
    int lvds_channel = 0;                                                              |
    uint32_t ipu_id, disp_id;                                                          |
                                                                                       |
    /* if input format not valid, make RGB666 as default*/                             |
    if (!valid_mode(setting->if_fmt)) {                                                |
        dev_warn(&ldb->pdev->dev, "Input pixel format not valid"                       |
                    " use default RGB666\n");                                          |
        setting->if_fmt = IPU_PIX_FMT_RGB666;                                          |
    }                                                                                  |
    //初始化                                                                           |
    if (!ldb->inited) {                                                                |
        char di_clk[] = "ipu1_di0_clk";                                                |
        char ldb_clk[] = "ldb_di0_clk";                                                |
                                                                                       |
        setting_idx = 0;                                                               |
        res = platform_get_resource(ldb->pdev, IORESOURCE_MEM, 0);                     |
        if (IS_ERR(res))                                                               |
            return -ENOMEM;                                                            |
                                                                                       |
        base_addr = res->start;                                                        |
        ldb->reg = ioremap(base_addr, res->end - res->start + 1);                      |
        ldb->control_reg = ldb->reg + 2;                                               |
        ldb->gpr3_reg = ldb->reg + 3;                                                  |
                                                                                       |
        ldb->lvds_bg_reg = regulator_get(&ldb->pdev->dev, plat_data->lvds_bg_reg);     |
        if (!IS_ERR(ldb->lvds_bg_reg)) {                                               |
            regulator_set_voltage(ldb->lvds_bg_reg, 2500000, 2500000);                 |
            regulator_enable(ldb->lvds_bg_reg);                                        |
        }                                                                              |
                                                                                       |
        /* ipu selected by platform data setting */                                    |
        setting->dev_id = plat_data->ipu_id;                                           |
                                                                                       |
        reg = readl(ldb->control_reg);                                                 |
        //ldb参考电阻选择                                                              |
        /* refrence resistor select */                                                 |
        reg &= ~LDB_BGREF_RMODE_MASK;                                                  |
        if (plat_data->ext_ref)                                                        |
            reg |= LDB_BGREF_RMODE_EXT;                                                |
        else                                                                           |
            reg |= LDB_BGREF_RMODE_INT;                                                |
        //使用SPWG标准对数据进行映射                                                   |
        /* TODO: now only use SPWG data mapping for both channel */                    |
        reg &= ~(LDB_BIT_MAP_CH0_MASK | LDB_BIT_MAP_CH1_MASK);                         |
        reg |= LDB_BIT_MAP_CH0_SPWG | LDB_BIT_MAP_CH1_SPWG;                            |
                                                                                       |
        /* channel mode setting */                                                     |
        reg &= ~(LDB_CH0_MODE_MASK | LDB_CH1_MODE_MASK);                               |
        reg &= ~(LDB_DATA_WIDTH_CH0_MASK | LDB_DATA_WIDTH_CH1_MASK);                   |
        //指定数据宽度，24bit或者18bit                                                 |
        if (bits_per_pixel(setting->if_fmt) == 24)                                     |
            reg |= LDB_DATA_WIDTH_CH0_24 | LDB_DATA_WIDTH_CH1_24;                      |
        else                                                                           |
            reg |= LDB_DATA_WIDTH_CH0_18 | LDB_DATA_WIDTH_CH1_18;                      |
                                                                                       |
        if (g_ldb_mode)                                                                |
            ldb->mode = g_ldb_mode;                                                    |
        else                                                                           |
            ldb->mode = plat_data->mode;                                               |
        //single mode                                                                  |
        if ((ldb->mode == LDB_SIN0) || (ldb->mode == LDB_SIN1)) {                      |
            ret = ldb->mode - LDB_SIN0;                                                |
            if (plat_data->disp_id != ret) {                                           |
                dev_warn(&ldb->pdev->dev,                                              |
                    "change IPU DI%d to IPU DI%d for LDB "                             |
                    "channel%d.\n",                                                    |
                    plat_data->disp_id, ret, ret);                                     |
                plat_data->disp_id = ret;                                              |
            }                                                                          |
        // separate mode                                                               |
        } else if (((ldb->mode == LDB_SEP0) || (ldb->mode == LDB_SEP1))                |
                && (cpu_is_mx6q() || cpu_is_mx6dl())) {                                |
            if (plat_data->disp_id == plat_data->sec_disp_id) {                        |
                dev_err(&ldb->pdev->dev,                                               |
                    "For LVDS separate mode,"                                          |
                    "two DIs should be different!\n");                                 |
                return -EINVAL;                                                        |
            }                                                                          |
                                                                                       |
            if (((!plat_data->disp_id) && (ldb->mode == LDB_SEP1))                     |
                || ((plat_data->disp_id) &&                                            |
                    (ldb->mode == LDB_SEP0))) {                                        |
                dev_dbg(&ldb->pdev->dev,                                               |
                    "LVDS separate mode:"                                              |
                    "swap DI configuration!\n");                                       |
                ipu_id = plat_data->ipu_id;                                            |
                disp_id = plat_data->disp_id;                                          |
                plat_data->ipu_id = plat_data->sec_ipu_id;                             |
                plat_data->disp_id = plat_data->sec_disp_id;                           |
                plat_data->sec_ipu_id = ipu_id;                                        |
                plat_data->sec_disp_id = disp_id;                                      |
            }                                                                          |
        }                                                                              |
        // split mode                                                                  |
        if (ldb->mode == LDB_SPL_DI0) {                                                |
            reg |= LDB_SPLIT_MODE_EN | LDB_CH0_MODE_EN_TO_DI0                          |
                | LDB_CH1_MODE_EN_TO_DI0;                                              |
            setting->disp_id = 0;                                                      |
        } else if (ldb->mode == LDB_SPL_DI1) {                                         |
            reg |= LDB_SPLIT_MODE_EN | LDB_CH0_MODE_EN_TO_DI1                          |
                | LDB_CH1_MODE_EN_TO_DI1;                                              |
            setting->disp_id = 1;                                                      |
        } else if (ldb->mode == LDB_DUL_DI0) {                                         |
            reg &= ~LDB_SPLIT_MODE_EN;                                                 |
            reg |= LDB_CH0_MODE_EN_TO_DI0 | LDB_CH1_MODE_EN_TO_DI0;                    |
            setting->disp_id = 0;                                                      |
        } else if (ldb->mode == LDB_DUL_DI1) {                                         |
            reg &= ~LDB_SPLIT_MODE_EN;                                                 |
            reg |= LDB_CH0_MODE_EN_TO_DI1 | LDB_CH1_MODE_EN_TO_DI1;                    |
            setting->disp_id = 1;                                                      |
        } else if (ldb->mode == LDB_SIN0) {                                            |
            reg &= ~LDB_SPLIT_MODE_EN;                                                 |
            setting->disp_id = plat_data->disp_id;                                     |
            if (setting->disp_id == 0)                                                 |
                reg |= LDB_CH0_MODE_EN_TO_DI0;                                         |
            else                                                                       |
                reg |= LDB_CH0_MODE_EN_TO_DI1;                                         |
            ch_mask = LDB_CH0_MODE_MASK;                                               |
            ch_val = reg & LDB_CH0_MODE_MASK;                                          |
        } else if (ldb->mode == LDB_SIN1) {                                            |
            reg &= ~LDB_SPLIT_MODE_EN;                                                 |
            setting->disp_id = plat_data->disp_id;                                     |
            if (setting->disp_id == 0)                                                 |
                reg |= LDB_CH1_MODE_EN_TO_DI0;                                         |
            else                                                                       |
                reg |= LDB_CH1_MODE_EN_TO_DI1;                                         |
            ch_mask = LDB_CH1_MODE_MASK;                                               |
            ch_val = reg & LDB_CH1_MODE_MASK;                                          |
        } else { /* separate mode*/                                                    |
        // dual mode                                                                   |
            setting->disp_id = plat_data->disp_id;                                     |
                                                                                       |
            /* first output is LVDS0 or LVDS1 */                                       |
            if (ldb->mode == LDB_SEP0)                                                 |
                lvds_channel = 0;                                                      |
            else                                                                       |
                lvds_channel = 1;                                                      |
                                                                                       |
            reg &= ~LDB_SPLIT_MODE_EN;                                                 |
                                                                                       |
            if ((lvds_channel == 0) && (setting->disp_id == 0))                        |
                reg |= LDB_CH0_MODE_EN_TO_DI0;                                         |
            else if ((lvds_channel == 0) && (setting->disp_id == 1))                   |
                reg |= LDB_CH0_MODE_EN_TO_DI1;                                         |
            else if ((lvds_channel == 1) && (setting->disp_id == 0))                   |
                reg |= LDB_CH1_MODE_EN_TO_DI0;                                         |
            else                                                                       |
                reg |= LDB_CH1_MODE_EN_TO_DI1;                                         |
            ch_mask = lvds_channel ? LDB_CH1_MODE_MASK :                               |
                    LDB_CH0_MODE_MASK;                                                 |
            ch_val = reg & ch_mask;                                                    |
                                                                                       |
            if (bits_per_pixel(setting->if_fmt) == 24) {                               |
                if (lvds_channel == 0)                                                 |
                    reg &= ~LDB_DATA_WIDTH_CH1_24;                                     |
                else                                                                   |
                    reg &= ~LDB_DATA_WIDTH_CH0_24;                                     |
            } else {                                                                   |
                if (lvds_channel == 0)                                                 |
                    reg &= ~LDB_DATA_WIDTH_CH1_18;                                     |
                else                                                                   |
                    reg &= ~LDB_DATA_WIDTH_CH0_18;                                     |
            }                                                                          |
        }                                                                              |
                                                                                       |
        writel(reg, ldb->control_reg);                                                 |
        if (ldb->mode <  LDB_SIN0) {                                                   |
            ch_mask = LDB_CH0_MODE_MASK | LDB_CH1_MODE_MASK;                           |
            ch_val = reg & (LDB_CH0_MODE_MASK | LDB_CH1_MODE_MASK);                    |
        }                                                                              |
                                                                                       |
        /* clock setting */                                                            |
        if ((cpu_is_mx6q() || cpu_is_mx6dl()) &&                                       |
            ((ldb->mode == LDB_SEP0) || (ldb->mode == LDB_SEP1)))                      |
            ldb_clk[6] += lvds_channel;                                                |
        else                                                                           |
            ldb_clk[6] += setting->disp_id;                                            |
        ldb->setting[setting_idx].ldb_di_clk = clk_get(&ldb->pdev->dev,                |
                                ldb_clk);                                              |
        if (IS_ERR(ldb->setting[setting_idx].ldb_di_clk)) {                            |
            dev_err(&ldb->pdev->dev, "get ldb clk0 failed\n");                         |
            iounmap(ldb->reg);                                                         |
            return PTR_ERR(ldb->setting[setting_idx].ldb_di_clk);                      |
        }                                                                              |
        di_clk[3] += setting->dev_id;                                                  |
        di_clk[7] += setting->disp_id;                                                 |
        ldb->setting[setting_idx].di_clk = clk_get(&ldb->pdev->dev,                    |
                                di_clk);                                               |
        if (IS_ERR(ldb->setting[setting_idx].di_clk)) {                                |
            dev_err(&ldb->pdev->dev, "get di clk0 failed\n");                          |
            iounmap(ldb->reg);                                                         |
            return PTR_ERR(ldb->setting[setting_idx].di_clk);                          |
        }                                                                              |
                                                                                       |
        dev_dbg(&ldb->pdev->dev, "ldb_clk to di clk: %s -> %s\n", ldb_clk, di_clk);    |
                                                                                       |
        /* fb notifier for clk setting */                                              |
        ldb->nb.notifier_call = ldb_fb_event,                                          |
        ret = fb_register_client(&ldb->nb);                                            |
        if (ret < 0) {                                                                 |
            iounmap(ldb->reg);                                                         |
            return ret;                                                                |
        }                                                                              |
                                                                                       |
        ldb->inited = true;                                                            |
                                                                                       |
        i2c_dev = ldb_i2c_client[lvds_channel];                                        |
                                                                                       |
    } else { /* second time for separate mode */                                       |
        char di_clk[] = "ipu1_di0_clk";                                                |
        char ldb_clk[] = "ldb_di0_clk";                                                |
                                                                                       |
        if ((ldb->mode == LDB_SPL_DI0) ||                                              |
            (ldb->mode == LDB_SPL_DI1) ||                                              |
            (ldb->mode == LDB_DUL_DI0) ||                                              |
            (ldb->mode == LDB_DUL_DI1) ||                                              |
            (ldb->mode == LDB_SIN0) ||                                                 |
            (ldb->mode == LDB_SIN1)) {                                                 |
            dev_err(&ldb->pdev->dev, "for second ldb disp"                             |
                    "ldb mode should in separate mode\n");                             |
            return -EINVAL;                                                            |
        }                                                                              |
                                                                                       |
        setting_idx = 1;                                                               |
        if (cpu_is_mx6q() || cpu_is_mx6dl()) {                                         |
            setting->dev_id = plat_data->sec_ipu_id;                                   |
            setting->disp_id = plat_data->sec_disp_id;                                 |
        } else {                                                                       |
            setting->dev_id = plat_data->ipu_id;                                       |
            setting->disp_id = !plat_data->disp_id;                                    |
        }                                                                              |
        if (setting->disp_id == ldb->setting[0].di) {                                  |
            dev_err(&ldb->pdev->dev, "Err: for second ldb disp in"                     |
                "separate mode, DI should be different!\n");                           |
            return -EINVAL;                                                            |
        }                                                                              |
                                                                                       |
        /* second output is LVDS0 or LVDS1 */                                          |
        if (ldb->mode == LDB_SEP0)                                                     |
            lvds_channel = 1;                                                          |
        else                                                                           |
            lvds_channel = 0;                                                          |
                                                                                       |
        reg = readl(ldb->control_reg);                                                 |
        if ((lvds_channel == 0) && (setting->disp_id == 0))                            |
            reg |= LDB_CH0_MODE_EN_TO_DI0;                                             |
        else if ((lvds_channel == 0) && (setting->disp_id == 1))                       |
            reg |= LDB_CH0_MODE_EN_TO_DI1;                                             |
        else if ((lvds_channel == 1) && (setting->disp_id == 0))                       |
            reg |= LDB_CH1_MODE_EN_TO_DI0;                                             |
        else                                                                           |
            reg |= LDB_CH1_MODE_EN_TO_DI1;                                             |
        ch_mask = lvds_channel ?  LDB_CH1_MODE_MASK :                                  |
                LDB_CH0_MODE_MASK;                                                     |
        ch_val = reg & ch_mask;                                                        |
                                                                                       |
        if (bits_per_pixel(setting->if_fmt) == 24) {                                   |
            if (lvds_channel == 0)                                                     |
                reg |= LDB_DATA_WIDTH_CH0_24;                                          |
            else                                                                       |
                reg |= LDB_DATA_WIDTH_CH1_24;                                          |
        } else {                                                                       |
            if (lvds_channel == 0)                                                     |
                reg |= LDB_DATA_WIDTH_CH0_18;                                          |
            else                                                                       |
                reg |= LDB_DATA_WIDTH_CH1_18;                                          |
        }                                                                              |
        writel(reg, ldb->control_reg);                                                 |
                                                                                       |
        /* clock setting */                                                            |
        if (cpu_is_mx6q() || cpu_is_mx6dl())                                           |
            ldb_clk[6] += lvds_channel;                                                |
        else                                                                           |
            ldb_clk[6] += setting->disp_id;                                            |
        ldb->setting[setting_idx].ldb_di_clk = clk_get(&ldb->pdev->dev,                |
                                ldb_clk);                                              |
        if (IS_ERR(ldb->setting[setting_idx].ldb_di_clk)) {                            |
            dev_err(&ldb->pdev->dev, "get ldb clk1 failed\n");                         |
            return PTR_ERR(ldb->setting[setting_idx].ldb_di_clk);                      |
        }                                                                              |
        di_clk[3] += setting->dev_id;                                                  |
        di_clk[7] += setting->disp_id;                                                 |
        ldb->setting[setting_idx].di_clk = clk_get(&ldb->pdev->dev,                    |
                                di_clk);                                               |
        if (IS_ERR(ldb->setting[setting_idx].di_clk)) {                                |
            dev_err(&ldb->pdev->dev, "get di clk1 failed\n");                          |
            return PTR_ERR(ldb->setting[setting_idx].di_clk);                          |
        }                                                                              |
                                                                                       |
        i2c_dev = ldb_i2c_client[lvds_channel];                                        |
                                                                                       |
        dev_dbg(&ldb->pdev->dev, "ldb_clk to di clk: %s -> %s\n", ldb_clk, di_clk);    |
    }                                                                                  |
                                                                                       |
    if (i2c_dev)                                                                       |
        mxc_dispdrv_setdev(ldb->disp_ldb, &i2c_dev->dev);                              |
    else                                                                               |
        mxc_dispdrv_setdev(ldb->disp_ldb, NULL);                                       |
                                                                                       |
    ldb->setting[setting_idx].ch_mask = ch_mask;                                       |
    ldb->setting[setting_idx].ch_val = ch_val;                                         |
                                                                                       |
    if (cpu_is_mx6q() || cpu_is_mx6dl())                                               |
        ldb_ipu_ldb_route(setting->dev_id, setting->disp_id, ldb);          -----------------+
                                                                                       |     |
    /*                                                                                 |     |
     * ldb_di0_clk -> ipux_di0_clk                                                     |     |
     * ldb_di1_clk -> ipux_di1_clk                                                     |     |
     */                                                                                |     |
    clk_set_parent(ldb->setting[setting_idx].di_clk,                                   |     |
            ldb->setting[setting_idx].ldb_di_clk);                                     |     |
                                                                                       |     |
    /* must use spec video mode defined by driver */                                   |     |
    ret = fb_find_mode(&setting->fbi->var, setting->fbi, setting->dft_mode_str,        |     |
                ldb_modedb, ldb_modedb_sz, NULL, setting->default_bpp);                |     |
    if (ret != 1)                                                                      |     |
        fb_videomode_to_var(&setting->fbi->var, &ldb_modedb[0]);                       |     |
                                                                                       |     |
    INIT_LIST_HEAD(&setting->fbi->modelist);                                           |     |
    for (i = 0; i < ldb_modedb_sz; i++) {                                              |     |
        struct fb_videomode m;                                                         |     |
        fb_var_to_videomode(&m, &setting->fbi->var);                                   |     |
        if (fb_mode_is_equal(&m, &ldb_modedb[i])) {                                    |     |
            fb_add_videomode(&ldb_modedb[i],                                           |     |
                    &setting->fbi->modelist);                                          |     |
            break;                                                                     |     |
        }                                                                              |     |
    }                                                                                  |     |
                                                                                       |     |
    /* get screen size in edid */                                                      |     |
    if (i2c_dev) {                                                                     |     |
        ret = mxc_ldb_edidread(i2c_dev);                                               |     |
        if (ret > 0) {                                                                 |     |
            fb_edid_to_monspecs(&g_edid[lvds_channel][0],                              |     |
                        &setting->fbi->monspecs);                                      |     |
            /* centimeter to millimeter */                                             |     |
            setting->fbi->var.width =                                                  |     |
                    setting->fbi->monspecs.max_x * 10;                                 |     |
            setting->fbi->var.height =                                                 |     |
                    setting->fbi->monspecs.max_y * 10;                                 |     |
        } else {                                                                       |     |
            /* ignore i2c access failure */                                            |     |
            ret = 0;                                                                   |     |
        }                                                                              |     |
    }                                                                                  |     |
                                                                                       |     |
    /* save current ldb setting for fb notifier */                                     |     |
    ldb->setting[setting_idx].active = true;                                           |     |
    ldb->setting[setting_idx].ipu = setting->dev_id;                                   |     |
    ldb->setting[setting_idx].di = setting->disp_id;                                   |     |
    return ret;                                                                        |     |
}                                                                                      |     |
                                                                                       |     |
static int ldb_disp_setup(struct mxc_dispdrv_handle *disp, struct fb_info *fbi)   <----+     |
{                                                                                            |
    uint32_t reg, val;                                                                       |
    uint32_t pixel_clk, rounded_pixel_clk;                                                   |
    struct clk *ldb_clk_parent;                                                              |
    struct ldb_data *ldb = mxc_dispdrv_getdata(disp);                                        |
    int setting_idx, di;                                                                     |
                                                                                             |
    setting_idx = find_ldb_setting(ldb, fbi);                                                |
    if (setting_idx < 0)                                                                     |
        return setting_idx;                                                                  |
                                                                                             |
    di = ldb->setting[setting_idx].di;                                                       |
                                                                                             |
    /* restore channel mode setting */                                                       |
    val = readl(ldb->control_reg);                                                           |
    val |= ldb->setting[setting_idx].ch_val;                                                 |
    writel(val, ldb->control_reg);                                                           |
    dev_dbg(&ldb->pdev->dev, "LDB setup, control reg:0x%x\n",                                |
            readl(ldb->control_reg));                                                        |
    //设置vsync信号的极性
    /* vsync setup */                                                                        |
    reg = readl(ldb->control_reg);                                                           |
    if (fbi->var.sync & FB_SYNC_VERT_HIGH_ACT) {                                             |
        if (di == 0)                                                                         |
            reg = (reg & ~LDB_DI0_VS_POL_MASK)                                               |
                | LDB_DI0_VS_POL_ACT_HIGH;                                                   |
        else                                                                                 |
            reg = (reg & ~LDB_DI1_VS_POL_MASK)                                               |
                | LDB_DI1_VS_POL_ACT_HIGH;                                                   |
    } else {                                                                                 |
        if (di == 0)                                                                         |
            reg = (reg & ~LDB_DI0_VS_POL_MASK)                                               |
                | LDB_DI0_VS_POL_ACT_LOW;                                                    |
        else                                                                                 |
            reg = (reg & ~LDB_DI1_VS_POL_MASK)                                               |
                | LDB_DI1_VS_POL_ACT_LOW;                                                    |
    }                                                                                        |
    writel(reg, ldb->control_reg);                                                           |
                                                                                             |
    /* clk setup */                                                                          |
    //时钟关闭                                                                               |
    if (ldb->setting[setting_idx].clk_en)                                                    |
        clk_disable(ldb->setting[setting_idx].ldb_di_clk);                                   |
    pixel_clk = (PICOS2KHZ(fbi->var.pixclock)) * 1000UL;                                     |
    ldb_clk_parent = clk_get_parent(ldb->setting[setting_idx].ldb_di_clk);                   |
    if ((ldb->mode == LDB_SPL_DI0) || (ldb->mode == LDB_SPL_DI1))                            |
        clk_set_rate(ldb_clk_parent, pixel_clk * 7 / 2);                                     |
    else                                                                                     |
        clk_set_rate(ldb_clk_parent, pixel_clk * 7);                                         |
    rounded_pixel_clk = clk_round_rate(ldb->setting[setting_idx].ldb_di_clk,                 |
            pixel_clk);                                                                      |
    clk_set_rate(ldb->setting[setting_idx].ldb_di_clk, rounded_pixel_clk);                   |
    //时钟使能                                                                               |
    clk_enable(ldb->setting[setting_idx].ldb_di_clk);                                        |
    if (!ldb->setting[setting_idx].clk_en)                                                   |
        ldb->setting[setting_idx].clk_en = true;                                             |
                                                                                             |
    return 0;                                                                                |
}                                                                                            |
                                                                                             |
#define LVDS_MUX_CTL_WIDTH    2                                                              |
#define LVDS_MUX_CTL_MASK    3                                                               |
#define LVDS0_MUX_CTL_OFFS    6                                                              |
#define LVDS1_MUX_CTL_OFFS    8                                                              |
#define LVDS0_MUX_CTL_MASK    (LVDS_MUX_CTL_MASK << 6)                                       |
#define LVDS1_MUX_CTL_MASK    (LVDS_MUX_CTL_MASK << 8)                                       |
#define ROUTE_IPU_DI(ipu, di)    (((ipu << 1) | di) & LVDS_MUX_CTL_MASK)                     |
static int ldb_ipu_ldb_route(int ipu, int di, struct ldb_data *ldb)              <-----------+
{
    uint32_t reg;
    int channel;
    int shift;
    int mode = ldb->mode;
    //IOMUXC_GPR3
    reg = readl(ldb->gpr3_reg);
    if (mode < LDB_SIN0) {
        reg &= ~(LVDS0_MUX_CTL_MASK | LVDS1_MUX_CTL_MASK);
        reg |= (ROUTE_IPU_DI(ipu, di) << LVDS0_MUX_CTL_OFFS) |
            (ROUTE_IPU_DI(ipu, di) << LVDS1_MUX_CTL_OFFS);
        dev_dbg(&ldb->pdev->dev,
            "Dual/Split mode both channels route to IPU%d-DI%d\n",
            ipu, di);
    } else if ((mode == LDB_SIN0) || (mode == LDB_SIN1)) {
        reg &= ~(LVDS0_MUX_CTL_MASK | LVDS1_MUX_CTL_MASK);      //选择lvds源
        channel = mode - LDB_SIN0;
        shift = LVDS0_MUX_CTL_OFFS + channel * LVDS_MUX_CTL_WIDTH;
        reg |= ROUTE_IPU_DI(ipu, di) << shift;
        dev_dbg(&ldb->pdev->dev,
            "Single mode channel %d route to IPU%d-DI%d\n",
                channel, ipu, di);
    } else {
        static bool first = true;

        if (first) {
            if (mode == LDB_SEP0) {
                reg &= ~LVDS0_MUX_CTL_MASK;
                channel = 0;
            } else {
                reg &= ~LVDS1_MUX_CTL_MASK;
                channel = 1;
            }
            first = false;
        } else {
            if (mode == LDB_SEP0) {
                reg &= ~LVDS1_MUX_CTL_MASK;
                channel = 1;
            } else {
                reg &= ~LVDS0_MUX_CTL_MASK;
                channel = 0;
            }
        }

        shift = LVDS0_MUX_CTL_OFFS + channel * LVDS_MUX_CTL_WIDTH;
        reg |= ROUTE_IPU_DI(ipu, di) << shift;

        dev_dbg(&ldb->pdev->dev,
            "Separate mode channel %d route to IPU%d-DI%d\n",
            channel, ipu, di);
    }
    writel(reg, ldb->gpr3_reg);

    return 0;
}
```

#### Author

Tony Liu

2016-8-31, Shenzhen