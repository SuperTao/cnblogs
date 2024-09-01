本文记录阅读linux ad7606驱动的笔记。

主要文件
 
drivers/staging/iio/adc/ad7606_spi.c
drivers/staging/iio/adc/ad7606_core.c
drivers/staging/iio/adc/ad7606_ring.c

```
drivers/staging/iio/adc/ad7606_spi.c

static int __init ad7606_spi_init(void)
{
    return spi_register_driver(&ad7606_driver);
}
module_init(ad7606_spi_init);

static struct spi_driver ad7606_driver = {
    .driver = {
        .name = "ad7606",
        .bus = &spi_bus_type,
        .owner = THIS_MODULE,
        .pm    = AD7606_SPI_PM_OPS,
    },
    .probe = ad7606_spi_probe,
    .remove = __devexit_p(ad7606_spi_remove),
    .id_table = ad7606_id,
};

static int __devinit ad7606_spi_probe(struct spi_device *spi)
{
    struct iio_dev *indio_dev;
    indio_dev = ad7606_probe(&spi->dev, spi->irq, NULL,   ------+
               spi_get_device_id(spi)->driver_data,             |
               &ad7606_spi_bops);     ---------+                |
                                               |                |
    if (IS_ERR(indio_dev))                     |                |
        return PTR_ERR(indio_dev);             |                |
                                               |                |
                                               |                |
    spi_set_drvdata(spi, indio_dev);           |                |
                                               |                |
    return 0;                                  |                |
}                                              |                |
                                               V                |
static const struct ad7606_bus_ops ad7606_spi_bops = {          |
    .read_block    = ad7606_spi_read_block,                     |
};                                                              |
                                                                |
drivers/staging/iio/adc/ad7606_core.c                           |
struct iio_dev *ad7606_probe(struct device *dev, int irq,   <---+
                  void __iomem *base_address,
                  unsigned id,
                  const struct ad7606_bus_ops *bops)
{
    struct ad7606_platform_data *pdata = dev->platform_data;
    struct ad7606_state *st;
    int ret, regdone = 0;
    struct iio_dev *indio_dev = iio_allocate_device(sizeof(*st));

    if (indio_dev == NULL) {
        ret = -ENOMEM;
        goto error_ret;
    }

    st = iio_priv(indio_dev);


    st->dev = dev;
    st->id = id;
    st->irq = irq;
    st->bops = bops;
    st->base_address = base_address;
    // 默认的模拟通道输入电压范围, 10V/5V
    st->range = pdata->default_range == 10000 ? 10000 : 5000;
    // 过采样率
    ret = ad7606_oversampling_get_index(pdata->default_os);
    if (ret < 0) {
        dev_warn(dev, "oversampling %d is not supported\n",
             pdata->default_os);
        st->oversampling = 0;
    } else {
        st->oversampling = pdata->default_os;
    }

    st->reg = regulator_get(dev, "vcc");
    if (!IS_ERR(st->reg)) {
        ret = regulator_enable(st->reg);
        if (ret)
            goto error_put_reg;
    }

    st->pdata = pdata;
    st->chip_info = &ad7606_chip_info_tbl[id];   -----------------------------+
                                                                              |
    indio_dev->dev.parent = dev;                                              |
    indio_dev->info = &ad7606_info;                                           |
    indio_dev->modes = INDIO_DIRECT_MODE;                                     |
    indio_dev->name = st->chip_info->name;                                    |
    indio_dev->channels = st->chip_info->channels;                            |
    indio_dev->num_channels = st->chip_info->num_channels;                    |
                                                                              |
    init_waitqueue_head(&st->wq_data_avail);                                  |
                                                                              |
    ret = ad7606_request_gpios(st);        -------------------------------+   |
    if (ret)                                                              |   |
        goto error_disable_reg;                                           |   |
                                                                          |   |
                                                                          |   |
    ret = ad7606_reset(st);                                               |   |
    if (ret)                                                              |   |
        dev_warn(st->dev, "failed to RESET: no RESET GPIO specified\n");  |   |
    // ad7606中断，我使用的是busy引脚作为中断输入。                       |   |
    ret = request_irq(st->irq, ad7606_interrupt,            -------+      |   |
        IRQF_TRIGGER_FALLING, st->chip_info->name, indio_dev);     |      |   |
    if (ret)                                                       |      |   |
        goto error_free_gpios;                                     |      |   |
                                                                   |      |   |
    ret = ad7606_register_ring_funcs_and_init(indio_dev);          |      |   |
    if (ret)                                                       |      |   |
        goto error_free_irq;                                       |      |   |
                                                                   |      |   |
    ret = iio_device_register(indio_dev);                          |      |   |
    if (ret)                                                       |      |   |
        goto error_free_irq;                                       |      |   |
    regdone = 1;                                                   |      |   |
                                                                   |      |   |
    ret = iio_ring_buffer_register_ex(indio_dev->ring, 0,          |      |   |
                      indio_dev->channels,                         |      |   |
                      indio_dev->num_channels);                    |      |   |
    if (ret)                                                       |      |   |
        goto error_cleanup_ring;                                   |      |   |
                                                                   |      |   |
    return indio_dev;                                              |      |   |
                                                                   |      |   |
error_cleanup_ring:                                                |      |   |
    ad7606_ring_cleanup(indio_dev);                                |      |   |
                                                                   |      |   |
error_free_irq:                                                    |      |   |
    free_irq(st->irq, indio_dev);                                  |      |   |
                                                                   |      |   |
error_free_gpios:                                                  |      |   |
    ad7606_free_gpios(st);                                         |      |   |
                                                                   |      |   |
error_disable_reg:                                                 |      |   |
    if (!IS_ERR(st->reg))                                          |      |   |
        regulator_disable(st->reg);                                |      |   |
error_put_reg:                                                     |      |   |
    if (!IS_ERR(st->reg))                                          |      |   |
        regulator_put(st->reg);                                    |      |   |
    if (regdone)                                                   |      |   |
        iio_device_unregister(indio_dev);                          |      |   |
    else                                                           |      |   |
        iio_free_device(indio_dev);                                |      |   |
error_ret:                                                         |      |   |
    return ERR_PTR(ret);                                           |      |   |
}                                                                  |      |   |
                                                                   |      |   |
// 中断处理函数                                                    |      |   |
static irqreturn_t ad7606_interrupt(int irq, void *dev_id)     <---+      |   |
{                                                                         |   |
    struct iio_dev *indio_dev = dev_id;                                   |   |
    struct ad7606_state *st = iio_priv(indio_dev);                        |   |
                                                                          |   |
    if (iio_ring_enabled(indio_dev)) {                                    |   |
        if (!work_pending(&st->poll_work))                                |   |
            schedule_work(&st->poll_work);                                |   |
    } else {                                                              |   |
        st->done = true;                                                  |   |
        // 唤醒中断                                                       |   |
        wake_up_interruptible(&st->wq_data_avail);                        |   |
    }                                                                     |   |
                                                                          |   |
    return IRQ_HANDLED;                                                   |   |
};                                                                        |   |
static int ad7606_request_gpios(struct ad7606_state *st)       <----------+   |
{                                                                             |
    struct gpio gpio_array[3] = {                                             |
        [0] = {                                                               |
            .gpio =  st->pdata->gpio_os0,                                     |
            .flags = GPIOF_DIR_OUT | ((st->oversampling & 1) ?                |
                 GPIOF_INIT_HIGH : GPIOF_INIT_LOW),                           |
            .label = "AD7606_OS0",                                            |
        },                                                                    |
        [1] = {                                                               |
            .gpio =  st->pdata->gpio_os1,                                     |
            .flags = GPIOF_DIR_OUT | ((st->oversampling & 2) ?                |
                 GPIOF_INIT_HIGH : GPIOF_INIT_LOW),                           |
            .label = "AD7606_OS1",                                            |
        },                                                                    |
        [2] = {                                                               |
            .gpio =  st->pdata->gpio_os2,                                     |
            .flags = GPIOF_DIR_OUT | ((st->oversampling & 4) ?                |
                 GPIOF_INIT_HIGH : GPIOF_INIT_LOW),                           |
            .label = "AD7606_OS2",                                            |
        },                                                                    |
    };                                                                        |
    int ret;                                                                  |
                                                                              |
    ret = gpio_request_one(st->pdata->gpio_convst, GPIOF_OUT_INIT_LOW,        |
                   "AD7606_CONVST");                                          |
    if (ret) {                                                                |
        dev_err(st->dev, "failed to request GPIO CONVST\n");                  |
        return ret;                                                           |
    }                                                                         |
                                                                              |
    ret = gpio_request_array(gpio_array, ARRAY_SIZE(gpio_array));             |
    if (!ret) {                                                               |
        st->have_os = true;                                                   |
    }                                                                         |
                                                                              |
    ret = gpio_request_one(st->pdata->gpio_reset, GPIOF_OUT_INIT_LOW,         |
                   "AD7606_RESET");                                           |
    if (!ret)                                                                 |
        st->have_reset = true;                                                |
                                                                              |
    ret = gpio_request_one(st->pdata->gpio_range, GPIOF_DIR_OUT |             |
                ((st->range == 10000) ? GPIOF_INIT_HIGH :                     |
                GPIOF_INIT_LOW), "AD7606_RANGE");                             |
    if (!ret)                                                                 |
        st->have_range = true;                                                |
                                                                              |
    ret = gpio_request_one(st->pdata->gpio_stby, GPIOF_OUT_INIT_HIGH,         |
                   "AD7606_STBY");                                            |
    if (!ret)                                                                 |
        st->have_stby = true;                                                 |
    // 是否定义了gpio_frstdata,没有定义就是-1.                                |
    // 我调试的时候这个信号有问题，所以就设置成-1.st->have_frstdata=false     |
    if (gpio_is_valid(st->pdata->gpio_frstdata)) {                            |
        ret = gpio_request_one(st->pdata->gpio_frstdata, GPIOF_IN,            |
                       "AD7606_FRSTDATA");                                    |
        if (!ret)                                                             |
            st->have_frstdata = true;                                         |
    }                                                                         |
                                                                              |
    return 0;                                                                 |
}                                                                             |
                                                                              |
static const struct ad7606_chip_info ad7606_chip_info_tbl[] = {        <------+
    /*
     * More devices added in future
     */
    [ID_AD7606_8] = {
        .name = "ad7606",
        .int_vref_mv = 2500,
        .channels = ad7606_8_channels,
        .num_channels = 8,
    },
    [ID_AD7606_6] = {
        .name = "ad7606-6",
        .int_vref_mv = 2500,
        .channels = ad7606_6_channels,
        .num_channels = 6,
    },
    [ID_AD7606_4] = {
        .name = "ad7606-4",
        .int_vref_mv = 2500,
        .channels = ad7606_4_channels,
        .num_channels = 4,
    },
};

static int ad7606_oversampling_get_index(unsigned val)
{
    unsigned char supported[] = {0, 2, 4, 8, 16, 32, 64};
    int i;

    for (i = 0; i < ARRAY_SIZE(supported); i++)
        if (val == supported[i])
            return i;

    return -EINVAL;
}

//应用层读取的时候会调用这个函数。
static int ad7606_read_raw(struct iio_dev *indio_dev,
               struct iio_chan_spec const *chan,
               int *val,
               int *val2,
               long m)
{
    int ret;
    struct ad7606_state *st = iio_priv(indio_dev);
    unsigned int scale_uv;

    switch (m) {
    case 0:
        mutex_lock(&indio_dev->mlock);
        if (iio_ring_enabled(indio_dev))
            ret = ad7606_scan_from_ring(indio_dev, chan->address);
        else
            ret = ad7606_scan_direct(indio_dev, chan->address);          ------+
        mutex_unlock(&indio_dev->mlock);                                       |
                                                                               |
        if (ret < 0)                                                           |
            return ret;                                                        |
        *val = (short) ret;                                                    |
        return IIO_VAL_INT;                                                    |
    case (1 << IIO_CHAN_INFO_SCALE_SHARED):                                    |
        scale_uv = (st->range * 1000 * 2)                                      |
            >> st->chip_info->channels[0].scan_type.realbits;                  |
        *val =  scale_uv / 1000;                                               |
        *val2 = (scale_uv % 1000) * 1000;                                      |
        return IIO_VAL_INT_PLUS_MICRO;                                         |
    }                                                                          |
    return -EINVAL;                                                            |
}                                                                              |
                                                                               |
static int ad7606_scan_direct(struct iio_dev *indio_dev, unsigned ch)     <----+
{
    struct ad7606_state *st = iio_priv(indio_dev);
    int ret;

    st->done = false;
    gpio_set_value(st->pdata->gpio_convst, 1);

    // 此处中断, 等待中断调用ad7606_interrupt函数唤醒中断。
    ret = wait_event_interruptible(st->wq_data_avail, st->done);
    if (ret)
        goto error_ret;
    // 判断have_frstdata是否为true
    if (st->have_frstdata) {
        ret = st->bops->read_block(st->dev, 1, st->data);
        if (ret)
            goto error_ret;
        if (!gpio_get_value(st->pdata->gpio_frstdata)) {
            /* This should never happen */
            ad7606_reset(st);
            ret = -EIO;
            goto error_ret;
        }
        ret = st->bops->read_block(st->dev,
            st->chip_info->num_channels - 1, &st->data[1]);
        if (ret){
            goto error_ret;
    } else {
        ret = st->bops->read_block(st->dev,
            st->chip_info->num_channels, st->data);
        if (ret)
            goto error_ret;
    }

    ret = st->data[ch];

error_ret:
    gpio_set_value(st->pdata->gpio_convst, 0);

    return ret;
}

static int ad7606_spi_read_block(struct device *dev,
                 int count, void *buf)
{
    struct spi_device *spi = to_spi_device(dev);
    int i, ret;
    unsigned short *data = buf;

    ret = spi_read(spi, (u8 *)buf, count * 2);
    if (ret < 0) {
        dev_err(&spi->dev, "SPI read error\n");
        return ret;
    }

    for (i = 0; i < count; i++) {
        data[i] = be16_to_cpu(data[i]);
    }

    return 0;
}


include/linux/spi/spi.h
static inline int
spi_read(struct spi_device *spi, void *buf, size_t len)
{
	struct spi_transfer	t = {
			.rx_buf		= buf,
			.len		= len,
		};
	struct spi_message	m;

	spi_message_init(&m);
	spi_message_add_tail(&t, &m);
	return spi_sync(spi, &m);
}
```

Tony Liu

2017-1-13, Shenzhen