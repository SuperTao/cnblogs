sound/soc/codecs/msm8x16-wcd.c
```
static struct spmi_device_id msm8x16_wcd_spmi_id_table[] = {
	{"wcd-spmi", MSM8X16_WCD_SPMI_DIGITAL},
	{"wcd-spmi", MSM8X16_WCD_SPMI_ANALOG},
	{}
};

static struct of_device_id msm8x16_wcd_spmi_of_match[] = {
	{ .compatible = "qcom,msm8x16_wcd_codec",},
	{ },
};

static struct spmi_driver wcd_spmi_driver = {
	.driver                 = {
		.owner          = THIS_MODULE,
		.name           = "wcd-spmi-core",
		.of_match_table = msm8x16_wcd_spmi_of_match
	},
#ifdef CONFIG_PM
		.suspend = msm8x16_wcd_spmi_suspend,
		.resume = msm8x16_wcd_spmi_resume,
#endif
	.id_table               = msm8x16_wcd_spmi_id_table,
	.probe                  = msm8x16_wcd_spmi_probe,
	.remove                 = msm8x16_wcd_spmi_remove,
};

static int __init msm8x16_wcd_codec_init(void)
{
	spmi_driver_register(&wcd_spmi_driver);
	return 0;
}

static int msm8x16_wcd_spmi_probe(struct spmi_device *spmi)
{
	int ret = 0;
	struct msm8x16_wcd *msm8x16 = NULL;
	struct msm8x16_wcd_pdata *pdata;
	struct resource *wcd_resource;
	int adsp_state;
	static int spmi_dev_registered_cnt;

	dev_dbg(&spmi->dev, "%s(%d):slave ID = 0x%x\n",
		__func__, __LINE__,  spmi->sid);

	adsp_state = apr_get_subsys_state();
	if (adsp_state != APR_SUBSYS_LOADED) {
		dev_dbg(&spmi->dev, "Adsp is not loaded yet %d\n",
				adsp_state);
		return -EPROBE_DEFER;
	}

	wcd_resource = spmi_get_resource(spmi, NULL, IORESOURCE_MEM, 0);
	if (!wcd_resource) {
		dev_err(&spmi->dev, "Unable to get Tombak base address\n");
		return -ENXIO;
	}

	switch (wcd_resource->start) {
	case TOMBAK_CORE_0_SPMI_ADDR:
		msm8x16_wcd_modules[0].spmi = spmi;
		msm8x16_wcd_modules[0].base = (spmi->sid << 16) +
						wcd_resource->start;
		wcd9xxx_spmi_set_dev(msm8x16_wcd_modules[0].spmi, 0);
		device_init_wakeup(&spmi->dev, true);
		break;
	case TOMBAK_CORE_1_SPMI_ADDR:
		msm8x16_wcd_modules[1].spmi = spmi;
		msm8x16_wcd_modules[1].base = (spmi->sid << 16) +
						wcd_resource->start;
		wcd9xxx_spmi_set_dev(msm8x16_wcd_modules[1].spmi, 1);
	if (wcd9xxx_spmi_irq_init()) {
		dev_err(&spmi->dev,
				"%s: irq initialization failed\n", __func__);
	} else {
		dev_dbg(&spmi->dev,
				"%s: irq initialization passed\n", __func__);
	}
		spmi_dev_registered_cnt++;
		goto register_codec;
	default:
		ret = -EINVAL;
		goto rtn;
	}


	dev_dbg(&spmi->dev, "%s(%d):start addr = 0x%pK\n",
		__func__, __LINE__,  &wcd_resource->start);

	if (wcd_resource->start != TOMBAK_CORE_0_SPMI_ADDR)
		goto rtn;

	if (spmi->dev.of_node) {
		dev_dbg(&spmi->dev, "%s:Platform data from device tree\n",
			__func__);
		// 解析设备树
		pdata = msm8x16_wcd_populate_dt_pdata(&spmi->dev);
		spmi->dev.platform_data = pdata;
	} else {
		dev_dbg(&spmi->dev, "%s:Platform data from board file\n",
			__func__);
		pdata = spmi->dev.platform_data;
	}
	if (pdata == NULL) {
		dev_err(&spmi->dev, "%s:Platform data failed to populate\n",
			__func__);
		goto rtn;
	}

	msm8x16 = kzalloc(sizeof(struct msm8x16_wcd), GFP_KERNEL);
	if (msm8x16 == NULL) {
		ret = -ENOMEM;
		goto rtn;
	}

	msm8x16->dev = &spmi->dev;
	msm8x16->read_dev = __msm8x16_wcd_reg_read;
	msm8x16->write_dev = __msm8x16_wcd_reg_write;
	ret = msm8x16_wcd_init_supplies(msm8x16, pdata);
	if (ret) {
		dev_err(&spmi->dev, "%s: Fail to enable Codec supplies\n",
			__func__);
		goto err_codec;
	}

	ret = msm8x16_wcd_enable_static_supplies(msm8x16, pdata);
	if (ret) {
		dev_err(&spmi->dev,
			"%s: Fail to enable Codec pre-reset supplies\n",
			   __func__);
		goto err_codec;
	}
	usleep_range(5, 6);

	ret = msm8x16_wcd_device_init(msm8x16);
	if (ret) {
		dev_err(&spmi->dev,
			"%s:msm8x16_wcd_device_init failed with error %d\n",
			__func__, ret);
		goto err_supplies;
	}
	dev_set_drvdata(&spmi->dev, msm8x16);
	spmi_dev_registered_cnt++;
register_codec:
	if ((spmi_dev_registered_cnt == MAX_MSM8X16_WCD_DEVICE) && (!ret)) {
		if (msm8x16_wcd_modules[0].spmi) {
			// 注册codec
			ret = snd_soc_register_codec(
					&msm8x16_wcd_modules[0].spmi->dev,
					&soc_codec_dev_msm8x16_wcd,
					msm8x16_wcd_i2s_dai,
					ARRAY_SIZE(msm8x16_wcd_i2s_dai));
			if (ret) {
				dev_err(&spmi->dev,
				"%s:snd_soc_register_codec failed with error %d\n",
				__func__, ret);
				goto err_supplies;
			}
		}
	}
	return ret;
err_supplies:
	msm8x16_wcd_disable_supplies(msm8x16, pdata);
err_codec:
	kfree(msm8x16);
rtn:
	return ret;
}

// codec操作函数
static struct snd_soc_codec_driver soc_codec_dev_msm8x16_wcd = {
	.probe	= msm8x16_wcd_codec_probe,
	.remove	= msm8x16_wcd_codec_remove,

	.read = msm8x16_wcd_read,
	.write = msm8x16_wcd_write,

	.suspend = msm8x16_wcd_suspend,
	.resume = msm8x16_wcd_resume,

	.readable_register = msm8x16_wcd_readable,
	.volatile_register = msm8x16_wcd_volatile,

	.reg_cache_size = MSM8X16_WCD_CACHE_SIZE,
	.reg_cache_default = msm8x16_wcd_reset_reg_defaults,
	.reg_word_size = 1,

	.controls = msm8x16_wcd_snd_controls,
	.num_controls = ARRAY_SIZE(msm8x16_wcd_snd_controls),
	.dapm_widgets = msm8x16_wcd_dapm_widgets,
	.num_dapm_widgets = ARRAY_SIZE(msm8x16_wcd_dapm_widgets),
	.dapm_routes = audio_map,
	.num_dapm_routes = ARRAY_SIZE(audio_map),
};

static struct snd_soc_dai_driver msm8x16_wcd_i2s_dai[] = {
	{
		.name = "msm8x16_wcd_i2s_rx1",
		.id = AIF1_PB,
		.playback = {
			.stream_name = "AIF1 Playback",
			.rates = MSM8X16_WCD_RATES,
			.formats = MSM8X16_WCD_FORMATS,
			.rate_max = 192000,
			.rate_min = 8000,
			.channels_min = 1,
			.channels_max = 3,
		},
		.ops = &msm8x16_wcd_dai_ops,
	},
	{
		.name = "msm8x16_wcd_i2s_tx1",
		.id = AIF1_CAP,
		.capture = {
			.stream_name = "AIF1 Capture",
			.rates = MSM8X16_WCD_RATES,
			.formats = MSM8X16_WCD_FORMATS,
			.rate_max = 192000,
			.rate_min = 8000,
			.channels_min = 1,
			.channels_max = 4,
		},
		.ops = &msm8x16_wcd_dai_ops,
	},
	{
		.name = "cajon_vifeedback",
		.id = AIF2_VIFEED,
		.capture = {
			.stream_name = "VIfeed",
			.rates = MSM8X16_WCD_RATES,
			.formats = MSM8X16_WCD_FORMATS,
			.rate_max = 48000,
			.rate_min = 48000,
			.channels_min = 2,
			.channels_max = 2,
		},
		.ops = &msm8x16_wcd_dai_ops,
	},
};
```


sound/soc/soc-core.c
```
int snd_soc_register_codec(struct device *dev,
			   const struct snd_soc_codec_driver *codec_drv,
			   struct snd_soc_dai_driver *dai_drv,
			   int num_dai)
{
	struct snd_soc_codec *codec;
	struct snd_soc_dai *dai;
	int ret, i;

	dev_dbg(dev, "codec register %s\n", dev_name(dev));

	codec = kzalloc(sizeof(struct snd_soc_codec), GFP_KERNEL);
	if (codec == NULL)
		return -ENOMEM;

	codec->component.dapm_ptr = &codec->dapm;
	codec->component.codec = codec;

	ret = snd_soc_component_initialize(&codec->component,
			&codec_drv->component_driver, dev);
	if (ret)
		goto err_free;
	// 初始化
	if (codec_drv->controls) {
		codec->component.controls = codec_drv->controls;
		codec->component.num_controls = codec_drv->num_controls;
	}
	if (codec_drv->dapm_widgets) {
		codec->component.dapm_widgets = codec_drv->dapm_widgets;
		codec->component.num_dapm_widgets = codec_drv->num_dapm_widgets;
	}
	if (codec_drv->dapm_routes) {
		codec->component.dapm_routes = codec_drv->dapm_routes;
		codec->component.num_dapm_routes = codec_drv->num_dapm_routes;
	}

	if (codec_drv->probe)
		codec->component.probe = snd_soc_codec_drv_probe;
	if (codec_drv->remove)
		codec->component.remove = snd_soc_codec_drv_remove;
	if (codec_drv->write)
		codec->component.write = snd_soc_codec_drv_write;
	if (codec_drv->read)
		codec->component.read = snd_soc_codec_drv_read;
	codec->volatile_register = codec_drv->volatile_register;
	codec->readable_register = codec_drv->readable_register;
	codec->writable_register = codec_drv->writable_register;
	codec->component.ignore_pmdown_time = codec_drv->ignore_pmdown_time;
	codec->dapm.idle_bias_off = codec_drv->idle_bias_off;
	codec->dapm.suspend_bias_off = codec_drv->suspend_bias_off;
	if (codec_drv->seq_notifier)
		codec->dapm.seq_notifier = codec_drv->seq_notifier;
	if (codec_drv->set_bias_level)
		codec->dapm.set_bias_level = snd_soc_codec_set_bias_level;
	codec->dev = dev;
	codec->driver = codec_drv;
	codec->component.val_bytes = codec_drv->reg_word_size;
	mutex_init(&codec->mutex);

#ifdef CONFIG_DEBUG_FS
	codec->component.init_debugfs = soc_init_codec_debugfs;
	codec->component.debugfs_prefix = "codec";
#endif

	if (codec_drv->get_regmap)
		codec->component.regmap = codec_drv->get_regmap(dev);

	for (i = 0; i < num_dai; i++) {
		fixup_codec_formats(&dai_drv[i].playback);
		fixup_codec_formats(&dai_drv[i].capture);
	}
	// 注册codec的dai
	ret = snd_soc_register_dais(&codec->component, dai_drv, num_dai, false);
	if (ret < 0) {
		dev_err(dev, "ASoC: Failed to regster DAIs: %d\n", ret);
		goto err_cleanup;
	}

	list_for_each_entry(dai, &codec->component.dai_list, list)
		dai->codec = codec;

	mutex_lock(&client_mutex);
	snd_soc_component_add_unlocked(&codec->component);
	list_add(&codec->list, &codec_list);
	mutex_unlock(&client_mutex);

	dev_dbg(codec->dev, "ASoC: Registered codec '%s'\n",
		codec->component.name);
	return 0;

err_cleanup:
	snd_soc_component_cleanup(&codec->component);
err_free:
	kfree(codec);
	return ret;
}
EXPORT_SYMBOL_GPL(snd_soc_register_codec);


static int snd_soc_register_dais(struct snd_soc_component *component,
	struct snd_soc_dai_driver *dai_drv, size_t count,
	bool legacy_dai_naming)
{
	struct device *dev = component->dev;
	struct snd_soc_dai *dai;
	unsigned int i;
	int ret;

	dev_dbg(dev, "ASoC: dai register %s #%Zu\n", dev_name(dev), count);

	component->dai_drv = dai_drv;
	component->num_dai = count;

	for (i = 0; i < count; i++) {

		dai = kzalloc(sizeof(struct snd_soc_dai), GFP_KERNEL);
		if (dai == NULL) {
			ret = -ENOMEM;
			goto err;
		}

		/*
		 * Back in the old days when we still had component-less DAIs,
		 * instead of having a static name, component-less DAIs would
		 * inherit the name of the parent device so it is possible to
		 * register multiple instances of the DAI. We still need to keep
		 * the same naming style even though those DAIs are not
		 * component-less anymore.
		 */
		if (count == 1 && legacy_dai_naming) {
			dai->name = fmt_single_name(dev, &dai->id);
		} else {
			dai->name = fmt_multiple_name(dev, &dai_drv[i]);
			if (dai_drv[i].id)
				dai->id = dai_drv[i].id;
			else
				dai->id = i;
		}
		if (dai->name == NULL) {
			kfree(dai);
			ret = -ENOMEM;
			goto err;
		}

		dai->component = component;
		dai->dev = dev;
		dai->driver = &dai_drv[i];
		if (!dai->driver->ops)
			dai->driver->ops = &null_dai_ops;

		list_add(&dai->list, &component->dai_list);

		dev_dbg(dev, "ASoC: Registered DAI '%s'\n", dai->name);
	}

	return 0;

err:
	snd_soc_unregister_dais(component);

	return ret;
}
```