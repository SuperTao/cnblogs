```

sound/soc/msm/msm8952.c

// 注册平台设备
static int __init msm8952_machine_init(void)
{
	return platform_driver_register(&msm8952_asoc_machine_driver);
}
late_initcall(msm8952_machine_init);

// 驱动和设备的匹配表
static const struct of_device_id msm8952_asoc_machine_of_match[]  = {
	{ .compatible = "qcom,msm8952-audio-codec", },
	{},
};

static struct platform_driver msm8952_asoc_machine_driver = {
	.driver = {
		.name = DRV_NAME,
		.owner = THIS_MODULE,
		.pm = &snd_soc_pm_ops,
		.of_match_table = msm8952_asoc_machine_of_match,
	},
	.probe = msm8952_asoc_machine_probe,
	.remove = msm8952_asoc_machine_remove,
};

static int msm8952_asoc_machine_probe(struct platform_device *pdev)
{
	struct snd_soc_card *card;
	struct msm8916_asoc_mach_data *pdata = NULL;
	const char *hs_micbias_type = "qcom,msm-hs-micbias-type";
	const char *ext_pa = "qcom,msm-ext-pa";
	const char *mclk = "qcom,msm-mclk-freq";
	const char *wsa = "asoc-wsa-codec-names";
	const char *wsa_prefix = "asoc-wsa-codec-prefixes";
	const char *type = NULL;
	const char *ext_pa_str = NULL;
	const char *wsa_str = NULL;
	const char *wsa_prefix_str = NULL;
	int num_strings;
	int ret, id, i, val;
	struct resource	*muxsel;
	char *temp_str = NULL;

	pdata = devm_kzalloc(&pdev->dev,
			sizeof(struct msm8916_asoc_mach_data), GFP_KERNEL);
	if (!pdata)
		return -ENOMEM;

	muxsel = platform_get_resource_byname(pdev, IORESOURCE_MEM,
			"csr_gp_io_mux_mic_ctl");
	if (!muxsel) {
		dev_err(&pdev->dev, "MUX addr invalid for MI2S\n");
		ret = -ENODEV;
		goto err1;
	}
	pdata->vaddr_gpio_mux_mic_ctl =
		ioremap(muxsel->start, resource_size(muxsel));
	if (pdata->vaddr_gpio_mux_mic_ctl == NULL) {
		pr_err("%s ioremap failure for muxsel virt addr\n",
				__func__);
		ret = -ENOMEM;
		goto err1;
	}

	muxsel = platform_get_resource_byname(pdev, IORESOURCE_MEM,
			"csr_gp_io_mux_spkr_ctl");
	if (!muxsel) {
		dev_err(&pdev->dev, "MUX addr invalid for MI2S\n");
		ret = -ENODEV;
		goto err;
	}
	pdata->vaddr_gpio_mux_spkr_ctl =
		ioremap(muxsel->start, resource_size(muxsel));
	if (pdata->vaddr_gpio_mux_spkr_ctl == NULL) {
		pr_err("%s ioremap failure for muxsel virt addr\n",
				__func__);
		ret = -ENOMEM;
		goto err;
	}

	muxsel = platform_get_resource_byname(pdev, IORESOURCE_MEM,
			"csr_gp_io_lpaif_pri_pcm_pri_mode_muxsel");
	if (!muxsel) {
		dev_err(&pdev->dev, "MUX addr invalid for MI2S\n");
		ret = -ENODEV;
		goto err;
	}
	pdata->vaddr_gpio_mux_pcm_ctl =
		ioremap(muxsel->start, resource_size(muxsel));
	if (pdata->vaddr_gpio_mux_pcm_ctl == NULL) {
		pr_err("%s ioremap failure for muxsel virt addr\n",
				__func__);
		ret = -ENOMEM;
		goto err;
	}

	muxsel = platform_get_resource_byname(pdev, IORESOURCE_MEM,
			"csr_gp_io_mux_quin_ctl");
	if (!muxsel) {
		dev_dbg(&pdev->dev, "MUX addr invalid for MI2S\n");
		goto parse_mclk_freq;
	}
	pdata->vaddr_gpio_mux_quin_ctl =
		ioremap(muxsel->start, resource_size(muxsel));
	if (pdata->vaddr_gpio_mux_quin_ctl == NULL) {
		pr_err("%s ioremap failure for muxsel virt addr\n",
				__func__);
		ret = -ENOMEM;
		goto err;
	}
parse_mclk_freq:
	// 获取时钟
	ret = of_property_read_u32(pdev->dev.of_node, mclk, &id);
	if (ret) {
		dev_err(&pdev->dev,
				"%s: missing %s in dt node\n", __func__, mclk);
		id = DEFAULT_MCLK_RATE;
	}
	pdata->mclk_freq = id;

	/*reading the gpio configurations from dtsi file*/
	ret = msm_gpioset_initialize(CLIENT_WCD_INT, &pdev->dev);
	if (ret < 0) {
		dev_err(&pdev->dev,
			"%s: error reading dtsi files%d\n", __func__, ret);
		goto err;
	}

	num_strings = of_property_count_strings(pdev->dev.of_node,
			wsa);
	if (num_strings > 0) {
		if (wsa881x_get_probing_count() < 2) {
			ret = -EPROBE_DEFER;
			goto err;
		} else if (wsa881x_get_presence_count() == num_strings) {
			bear_card.aux_dev = msm8952_aux_dev;
			bear_card.num_aux_devs = num_strings;
			bear_card.codec_conf = msm8952_codec_conf;
			bear_card.num_configs = num_strings;

			for (i = 0; i < num_strings; i++) {
				ret = of_property_read_string_index(
						pdev->dev.of_node, wsa,
						i, &wsa_str);
				if (ret) {
					dev_err(&pdev->dev,
						"%s:of read string %s i %d error %d\n",
						__func__, wsa, i, ret);
					goto err;
				}
				temp_str = kstrdup(wsa_str, GFP_KERNEL);
				if (!temp_str) {
					ret = -ENOMEM;
					goto err;
				}
				msm8952_aux_dev[i].codec_name = temp_str;
				temp_str = NULL;

				temp_str = kstrdup(wsa_str, GFP_KERNEL);
				if (!temp_str) {
					ret = -ENOMEM;
					goto err;
				}
				msm8952_codec_conf[i].dev_name = temp_str;
				temp_str = NULL;

				ret = of_property_read_string_index(
						pdev->dev.of_node, wsa_prefix,
						i, &wsa_prefix_str);
				if (ret) {
					dev_err(&pdev->dev,
						"%s:of read string %s i %d error %d\n",
						__func__, wsa_prefix, i, ret);
					goto err;
				}
				temp_str = kstrdup(wsa_prefix_str, GFP_KERNEL);
				if (!temp_str) {
					ret = -ENOMEM;
					goto err;
				}
				msm8952_codec_conf[i].name_prefix = temp_str;
				temp_str = NULL;
			}

			ret = msm8952_init_wsa_switch_supply(pdev, pdata);
			if (ret < 0) {
				pr_err("%s: failed to init wsa_switch vdd supply %d\n",
						__func__, ret);
				goto err;
			}
			wsa881x_set_mclk_callback(msm8952_enable_wsa_mclk);
			/* update the internal speaker boost usage */
			msm8x16_update_int_spk_boost(false);
		}
	}
	// 获取dai_links
	card = msm8952_populate_sndcard_dailinks(&pdev->dev);
	dev_info(&pdev->dev, "default codec configured\n");
	num_strings = of_property_count_strings(pdev->dev.of_node,
			ext_pa);
	if (num_strings < 0) {
		dev_err(&pdev->dev,
				"%s: missing %s in dt node or length is incorrect\n",
				__func__, ext_pa);
		goto err;
	}
	for (i = 0; i < num_strings; i++) {
		ret = of_property_read_string_index(pdev->dev.of_node,
				ext_pa, i, &ext_pa_str);
		if (ret) {
			dev_err(&pdev->dev, "%s:of read string %s i %d error %d\n",
					__func__, ext_pa, i, ret);
			goto err;
		}
		if (!strcmp(ext_pa_str, "primary"))
			pdata->ext_pa = (pdata->ext_pa | PRI_MI2S_ID);
		else if (!strcmp(ext_pa_str, "secondary"))
			pdata->ext_pa = (pdata->ext_pa | SEC_MI2S_ID);
		else if (!strcmp(ext_pa_str, "tertiary"))
			pdata->ext_pa = (pdata->ext_pa | TER_MI2S_ID);
		else if (!strcmp(ext_pa_str, "quaternary"))
			pdata->ext_pa = (pdata->ext_pa | QUAT_MI2S_ID);
		else if (!strcmp(ext_pa_str, "quinary"))
			pdata->ext_pa = (pdata->ext_pa | QUIN_MI2S_ID);
	}
	pr_debug("%s: ext_pa = %d\n", __func__, pdata->ext_pa);

	ret = is_us_eu_switch_gpio_support(pdev, pdata);
	if (ret < 0) {
		pr_err("%s: failed to is_us_eu_switch_gpio_support %d\n",
				__func__, ret);
		goto err;
	}

	ret = is_ext_spk_gpio_support(pdev, pdata);
	if (ret < 0)
		pr_err("%s:  doesn't support external speaker pa\n",
				__func__);

		get_dev_by_boardid(model_name);
		if(0 == strcmp(model_name,"eda71")){
			enable_aw8738 = of_property_read_bool(pdev->dev.of_node , "action,enable-aw8738");
			if(enable_aw8738)
			{
				gpio_request(pdata->spk_ext_pa_gpio, "AW8738_EN");
				gpio_direction_output(pdata->spk_ext_pa_gpio,0);
			}
		}
		//printk("enable_aw8738:%d model_name:%s.\n",enable_aw8738,model_name);
	// 查看micbias类型
	ret = of_property_read_string(pdev->dev.of_node,
		hs_micbias_type, &type);
	if (ret) {
		dev_err(&pdev->dev, "%s: missing %s in dt node\n",
			__func__, hs_micbias_type);
		goto err;
	}
	// 使用内部还是外部的micbias
	if (!strcmp(type, "external")) {
		dev_dbg(&pdev->dev, "Headset is using external micbias\n");
		mbhc_cfg.hs_ext_micbias = true;
	} else {
		dev_dbg(&pdev->dev, "Headset is using internal micbias\n");
		mbhc_cfg.hs_ext_micbias = false;
	}

	ret = of_property_read_u32(pdev->dev.of_node,
				  "qcom,msm-afe-clk-ver", &val);
	if (ret)
		pdata->afe_clk_ver = AFE_CLK_VERSION_V2;
	else
		pdata->afe_clk_ver = val;
	/* initialize the mclk */
	pdata->digital_cdc_clk.i2s_cfg_minor_version =
					AFE_API_VERSION_I2S_CONFIG;
	pdata->digital_cdc_clk.clk_val = pdata->mclk_freq;
	pdata->digital_cdc_clk.clk_root = 5;
	pdata->digital_cdc_clk.reserved = 0;
	/* initialize the digital codec core clk */
	pdata->digital_cdc_core_clk.clk_set_minor_version =
			AFE_API_VERSION_I2S_CONFIG;
	pdata->digital_cdc_core_clk.clk_id =
			Q6AFE_LPASS_CLK_ID_INTERNAL_DIGITAL_CODEC_CORE;
	pdata->digital_cdc_core_clk.clk_freq_in_hz =
			pdata->mclk_freq;
	pdata->digital_cdc_core_clk.clk_attri =
			Q6AFE_LPASS_CLK_ATTRIBUTE_COUPLE_NO;
	pdata->digital_cdc_core_clk.clk_root =
			Q6AFE_LPASS_CLK_ROOT_DEFAULT;
	pdata->digital_cdc_core_clk.enable = 1;

	/* Initialize loopback mode to false */
	pdata->lb_mode = false;

	msm8952_dt_parse_cap_info(pdev, pdata);

	card->dev = &pdev->dev;
	platform_set_drvdata(pdev, card);
	snd_soc_card_set_drvdata(card, pdata);
	// 解析设备数中声卡的名称, qcom,model = "msm8953-snd-card-mtp";
	ret = snd_soc_of_parse_card_name(card, "qcom,model");
	if (ret)
		goto err;
	/* initialize timer */
	INIT_DELAYED_WORK(&pdata->disable_mclk_work, msm8952_disable_mclk);
	mutex_init(&pdata->cdc_mclk_mutex);
	atomic_set(&pdata->mclk_rsc_ref, 0);
	if (card->aux_dev) {
		mutex_init(&pdata->wsa_mclk_mutex);
		atomic_set(&pdata->wsa_mclk_rsc_ref, 0);
	}
	atomic_set(&pdata->mclk_enabled, false);
	atomic_set(&quat_mi2s_clk_ref, 0);
	atomic_set(&quin_mi2s_clk_ref, 0);
	atomic_set(&auxpcm_mi2s_clk_ref, 0);
	// 声卡的routing
	ret = snd_soc_of_parse_audio_routing(card,
			"qcom,audio-routing");
	if (ret)
		goto err;

	ret = msm8952_populate_dai_link_component_of_node(card);
	if (ret) {
		ret = -EPROBE_DEFER;
		goto err;
	}
	// 注册声卡
	ret = snd_soc_register_card(card);
	if (ret) {
		dev_err(&pdev->dev, "snd_soc_register_card failed (%d)\n",
			ret);
		goto err;
	}
	return 0;
err:
	if (pdata->vaddr_gpio_mux_spkr_ctl)
		iounmap(pdata->vaddr_gpio_mux_spkr_ctl);
	if (pdata->vaddr_gpio_mux_mic_ctl)
		iounmap(pdata->vaddr_gpio_mux_mic_ctl);
	if (pdata->vaddr_gpio_mux_pcm_ctl)
		iounmap(pdata->vaddr_gpio_mux_pcm_ctl);
	if (pdata->vaddr_gpio_mux_quin_ctl)
		iounmap(pdata->vaddr_gpio_mux_quin_ctl);
	if (bear_card.num_aux_devs > 0) {
		for (i = 0; i < bear_card.num_aux_devs; i++) {
			kfree(msm8952_aux_dev[i].codec_name);
			kfree(msm8952_codec_conf[i].dev_name);
			kfree(msm8952_codec_conf[i].name_prefix);
		}
	}
err1:
	devm_kfree(&pdev->dev, pdata);
	return ret;
}

static struct snd_soc_card *msm8952_populate_sndcard_dailinks(
						struct device *dev)
{
	struct snd_soc_card *card = &bear_card;
	struct snd_soc_dai_link *dailink;
	int len1;

	card->name = dev_name(dev);
	len1 = ARRAY_SIZE(msm8952_dai);
	// dai_links复制给snd_soc_card, 注册的时候会用到
	memcpy(msm8952_dai_links, msm8952_dai, sizeof(msm8952_dai));
	dailink = msm8952_dai_links;
	if (of_property_read_bool(dev->of_node,
				"qcom,hdmi-dba-codec-rx")) {
		dev_dbg(dev, "%s(): hdmi audio support present\n",
				__func__);
		memcpy(dailink + len1, msm8952_hdmi_dba_dai_link,
				sizeof(msm8952_hdmi_dba_dai_link));
		len1 += ARRAY_SIZE(msm8952_hdmi_dba_dai_link);
	} else {
		dev_dbg(dev, "%s(): No hdmi dba present, add quin dai\n",
				__func__);
		memcpy(dailink + len1, msm8952_quin_dai_link,
				sizeof(msm8952_quin_dai_link));
		len1 += ARRAY_SIZE(msm8952_quin_dai_link);
	}
	if (of_property_read_bool(dev->of_node,
				"qcom,split-a2dp")) {
		dev_dbg(dev, "%s(): split a2dp support present\n",
				__func__);
		memcpy(dailink + len1, msm8952_split_a2dp_dai_link,
				sizeof(msm8952_split_a2dp_dai_link));
		len1 += ARRAY_SIZE(msm8952_split_a2dp_dai_link);
	}
	card->dai_link = dailink;
	card->num_links = len1;
	return card;
}



// machine驱动会用这些参数去匹配platform,codec,adi.
// 都是在platform，codec中定义
/* Digital audio interface glue - connects codec <---> CPU */
static struct snd_soc_dai_link msm8952_dai[] = {
	/* FrontEnd DAI Links */
	{/* hw:x,0 */
		.name = "MSM8952 Media1",		// Media1 播放链路
		.stream_name = "MultiMedia1",	// 匹配pcm id
		.cpu_dai_name	= "MultiMedia1",	// 匹配cpu dai driver
		.platform_name  = "msm-pcm-dsp.0",	// 匹配platform driver
		.dynamic = 1,
		.async_ops = ASYNC_DPCM_SND_SOC_PREPARE,
		.dpcm_playback = 1,
		.dpcm_capture = 1,
		.trigger = {SND_SOC_DPCM_TRIGGER_POST,
			SND_SOC_DPCM_TRIGGER_POST},
		.codec_dai_name = "snd-soc-dummy-dai",	// 匹配codec dai driver
		.codec_name = "snd-soc-dummy",	// 匹配codec driver
		.ignore_suspend = 1,
		/* this dainlink has playback support */
		.ignore_pmdown_time = 1,
		.be_id = MSM_FRONTEND_DAI_MULTIMEDIA1
	},
	{/* hw:x,1 */
		.name = "MSM8952 Media2",		// Media2播放链路
		.stream_name = "MultiMedia2",
		.cpu_dai_name   = "MultiMedia2",
		.platform_name  = "msm-pcm-dsp.0",
		.dynamic = 1,
		.dpcm_playback = 1,
		.dpcm_capture = 1,
		.codec_dai_name = "snd-soc-dummy-dai",
		.codec_name = "snd-soc-dummy",
		.trigger = {SND_SOC_DPCM_TRIGGER_POST,
			SND_SOC_DPCM_TRIGGER_POST},
		.ignore_suspend = 1,
		/* this dainlink has playback support */
		.ignore_pmdown_time = 1,
		.be_id = MSM_FRONTEND_DAI_MULTIMEDIA2,
	},
	{/* hw:x,2 */
		.name = "Circuit-Switch Voice",
		.stream_name = "CS-Voice",
		.cpu_dai_name   = "CS-VOICE",
		.platform_name  = "msm-pcm-voice",
		.dynamic = 1,
		.dpcm_playback = 1,
		.dpcm_capture = 1,
		.trigger = {SND_SOC_DPCM_TRIGGER_POST,
			SND_SOC_DPCM_TRIGGER_POST},
		.no_host_mode = SND_SOC_DAI_LINK_NO_HOST,
		.ignore_suspend = 1,
		/* this dainlink has playback support */
		.ignore_pmdown_time = 1,
		.be_id = MSM_FRONTEND_DAI_CS_VOICE,
		.codec_dai_name = "snd-soc-dummy-dai",
		.codec_name = "snd-soc-dummy",
	},
	...
}
```