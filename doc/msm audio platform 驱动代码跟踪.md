```
sound/soc/soc-core.c

static int __init snd_soc_init(void)
{
#ifdef CONFIG_DEBUG_FS
	snd_soc_debugfs_root = debugfs_create_dir("asoc", NULL);
	if (IS_ERR(snd_soc_debugfs_root) || !snd_soc_debugfs_root) {
		pr_warn("ASoC: Failed to create debugfs directory\n");
		snd_soc_debugfs_root = NULL;
	}

	if (!debugfs_create_file("codecs", 0444, snd_soc_debugfs_root, NULL,
				 &codec_list_fops))
		pr_warn("ASoC: Failed to create CODEC list debugfs file\n");

	if (!debugfs_create_file("dais", 0444, snd_soc_debugfs_root, NULL,
				 &dai_list_fops))
		pr_warn("ASoC: Failed to create DAI list debugfs file\n");

	if (!debugfs_create_file("platforms", 0444, snd_soc_debugfs_root, NULL,
				 &platform_list_fops))
		pr_warn("ASoC: Failed to create platform list debugfs file\n");
#endif

	snd_soc_util_init();

	return platform_driver_register(&soc_driver);
}
module_init(snd_soc_init);

const struct dev_pm_ops snd_soc_pm_ops = {
	.suspend = snd_soc_suspend,
	.resume = snd_soc_resume,
	.freeze = snd_soc_suspend,
	.thaw = snd_soc_resume,
	.poweroff = snd_soc_poweroff,
	.restore = snd_soc_resume,
};
EXPORT_SYMBOL_GPL(snd_soc_pm_ops);

/* ASoC platform driver */
static struct platform_driver soc_driver = {
	.driver		= {
		.name		= "soc-audio",	// 声卡设备
		.owner		= THIS_MODULE,
		.pm		= &snd_soc_pm_ops,
	},
	.probe		= soc_probe,
	.remove		= soc_remove,
};

static int soc_probe(struct platform_device *pdev)
{
	// 获取声卡
	struct snd_soc_card *card = platform_get_drvdata(pdev);

	/*
	 * no card, so machine driver should be registering card
	 * we should not be here in that case so ret error
	 */
	if (!card)
		return -EINVAL;

	dev_warn(&pdev->dev,
		 "ASoC: machine %s should use snd_soc_register_card()\n",
		 card->name);

	/* Bodge while we unpick instantiation */
	card->dev = &pdev->dev;
	// 注册声卡
	return snd_soc_register_card(card);
}

int snd_soc_register_card(struct snd_soc_card *card)
{
	int i, j, ret;

	if (!card->name || !card->dev)
		return -EINVAL;

	// 遍历dai_link的成员
	for (i = 0; i < card->num_links; i++) {
		struct snd_soc_dai_link *link = &card->dai_link[i];
		// 初始化codec
		ret = snd_soc_init_multicodec(card, link);
		if (ret) {
			dev_err(card->dev, "ASoC: failed to init multicodec\n");
			return ret;
		}

		for (j = 0; j < link->num_codecs; j++) {
			/*
			 * Codec must be specified by 1 of name or OF node,
			 * not both or neither.
			 */
			if (!!link->codecs[j].name ==
			    !!link->codecs[j].of_node) {
				dev_err(card->dev, "ASoC: Neither/both codec name/of_node are set for %s\n",
					link->name);
				return -EINVAL;
			}
			/* Codec DAI name must be specified */
			if (!link->codecs[j].dai_name) {
				dev_err(card->dev, "ASoC: codec_dai_name not set for %s\n",
					link->name);
				return -EINVAL;
			}
		}

		/*
		 * Platform may be specified by either name or OF node, but
		 * can be left unspecified, and a dummy platform will be used.
		 */
		if (link->platform_name && link->platform_of_node) {
			dev_err(card->dev,
				"ASoC: Both platform name/of_node are set for %s\n",
				link->name);
			return -EINVAL;
		}

		/*
		 * CPU device may be specified by either name or OF node, but
		 * can be left unspecified, and will be matched based on DAI
		 * name alone..
		 */
		if (link->cpu_name && link->cpu_of_node) {
			dev_err(card->dev,
				"ASoC: Neither/both cpu name/of_node are set for %s\n",
				link->name);
			return -EINVAL;
		}
		/*
		 * At least one of CPU DAI name or CPU device name/node must be
		 * specified
		 */
		if (!link->cpu_dai_name &&
		    !(link->cpu_name || link->cpu_of_node)) {
			dev_err(card->dev,
				"ASoC: Neither cpu_dai_name nor cpu_name/of_node are set for %s\n",
				link->name);
			return -EINVAL;
		}
	}
	// 设置声卡驱动信息
	dev_set_drvdata(card->dev, card);
	// 声卡初始化列表, 初始几个链表
	snd_soc_initialize_card_lists(card);

	soc_init_card_debugfs(card);
	// 分配空间
	card->rtd = devm_kzalloc(card->dev,
				 sizeof(struct snd_soc_pcm_runtime) *
				 (card->num_links + card->num_aux_devs),
				 GFP_KERNEL);
	if (card->rtd == NULL)
		return -ENOMEM;
	card->num_rtd = 0;
	card->rtd_aux = &card->rtd[card->num_links];

	for (i = 0; i < card->num_links; i++) {
		card->rtd[i].card = card;
		card->rtd[i].dai_link = &card->dai_link[i];
		card->rtd[i].codec_dais = devm_kzalloc(card->dev,
					sizeof(struct snd_soc_dai *) *
					(card->rtd[i].dai_link->num_codecs),
					GFP_KERNEL);
		if (card->rtd[i].codec_dais == NULL)
			return -ENOMEM;
	}

	for (i = 0; i < card->num_aux_devs; i++)
		card->rtd_aux[i].card = card;

	INIT_LIST_HEAD(&card->dapm_dirty);
	card->instantiated = 0;
	mutex_init(&card->mutex);
	mutex_init(&card->dapm_mutex);
	mutex_init(&card->dapm_power_mutex);
	// 初始化声卡
	ret = snd_soc_instantiate_card(card);
	if (ret != 0)
		soc_cleanup_card_debugfs(card);

	/* deactivate pins to sleep state */
	for (i = 0; i < card->num_rtd; i++) {
		struct snd_soc_pcm_runtime *rtd = &card->rtd[i];
		struct snd_soc_dai *cpu_dai = rtd->cpu_dai;
		int j;

		for (j = 0; j < rtd->num_codecs; j++) {
			struct snd_soc_dai *codec_dai = rtd->codec_dais[j];
			if (!codec_dai->active)
				pinctrl_pm_select_sleep_state(codec_dai->dev);
		}

		if (!cpu_dai->active)
			pinctrl_pm_select_sleep_state(cpu_dai->dev);
	}

	return ret;
}
EXPORT_SYMBOL_GPL(snd_soc_register_card);

static int snd_soc_instantiate_card(struct snd_soc_card *card)
{
	struct snd_soc_codec *codec;
	struct snd_soc_dai_link *dai_link;
	int ret, i, order, dai_fmt;

	mutex_lock_nested(&card->mutex, SND_SOC_CARD_CLASS_INIT);

	/* bind DAIs */
	// 绑定DAI 
	for (i = 0; i < card->num_links; i++) {
		ret = soc_bind_dai_link(card, i);
		if (ret != 0)
			goto base_error;
	}

	/* bind aux_devs too */
	// 绑定AUX
	for (i = 0; i < card->num_aux_devs; i++) {
		ret = soc_bind_aux_dev(card, i);
		if (ret != 0)
			goto base_error;
	}

	/* initialize the register cache for each available codec */
	list_for_each_entry(codec, &codec_list, list) {
		if (codec->cache_init)
			continue;
		// 初始化codec缓存
		ret = snd_soc_init_codec_cache(codec);
		if (ret < 0)
			goto base_error;
	}

	/* card bind complete so register a sound card */
	// 注册声卡
	ret = snd_card_new(card->dev, SNDRV_DEFAULT_IDX1, SNDRV_DEFAULT_STR1,
			card->owner, 0, &card->snd_card);
	if (ret < 0) {
		dev_err(card->dev,
			"ASoC: can't create sound card for card %s: %d\n",
			card->name, ret);
		goto base_error;
	}

	card->dapm.bias_level = SND_SOC_BIAS_OFF;
	card->dapm.dev = card->dev;
	card->dapm.card = card;
	list_add(&card->dapm.list, &card->dapm_list);

#ifdef CONFIG_DEBUG_FS
	snd_soc_dapm_debugfs_init(&card->dapm, card->debugfs_card_root);
#endif

#ifdef CONFIG_PM_SLEEP
	/* deferred resume work */
	INIT_WORK(&card->deferred_resume_work, soc_resume_deferred);
#endif

	if (card->dapm_widgets)
		snd_soc_dapm_new_controls(&card->dapm, card->dapm_widgets,
					  card->num_dapm_widgets);

	/* initialise the sound card only once */
	if (card->probe) {
		ret = card->probe(card);
		if (ret < 0)
			goto card_probe_error;
	}

	/* probe all components used by DAI links on this card */
	for (order = SND_SOC_COMP_ORDER_FIRST; order <= SND_SOC_COMP_ORDER_LAST;
			order++) {
		for (i = 0; i < card->num_links; i++) {
			ret = soc_probe_link_components(card, i, order);
			if (ret < 0) {
				dev_err(card->dev,
					"ASoC: failed to instantiate card %d\n",
					ret);
				goto probe_dai_err;
			}
		}
	}

	/* probe all DAI links on this card */
	for (order = SND_SOC_COMP_ORDER_FIRST; order <= SND_SOC_COMP_ORDER_LAST;
			order++) {
		for (i = 0; i < card->num_links; i++) {
			ret = soc_probe_link_dais(card, i, order);
			if (ret < 0) {
				dev_err(card->dev,
					"ASoC: failed to instantiate card %d\n",
					ret);
				goto probe_dai_err;
			}
		}
	}

	for (i = 0; i < card->num_aux_devs; i++) {
		ret = soc_probe_aux_dev(card, i);
		if (ret < 0) {
			dev_err(card->dev,
				"ASoC: failed to add auxiliary devices %d\n",
				ret);
			goto probe_aux_dev_err;
		}
	}

	snd_soc_dapm_link_dai_widgets(card);
	snd_soc_dapm_connect_dai_link_widgets(card);

	if (card->controls)
		snd_soc_add_card_controls(card, card->controls, card->num_controls);

	if (card->dapm_routes)
		snd_soc_dapm_add_routes(&card->dapm, card->dapm_routes,
					card->num_dapm_routes);

	for (i = 0; i < card->num_links; i++) {
		struct snd_soc_pcm_runtime *rtd = &card->rtd[i];
		dai_link = &card->dai_link[i];
		dai_fmt = dai_link->dai_fmt;

		if (dai_fmt) {
			struct snd_soc_dai **codec_dais = rtd->codec_dais;
			int j;

			for (j = 0; j < rtd->num_codecs; j++) {
				struct snd_soc_dai *codec_dai = codec_dais[j];

				ret = snd_soc_dai_set_fmt(codec_dai, dai_fmt);
				if (ret != 0 && ret != -ENOTSUPP)
					dev_warn(codec_dai->dev,
						 "ASoC: Failed to set DAI format: %d\n",
						 ret);
			}
		}

		/* If this is a regular CPU link there will be a platform */
		if (dai_fmt &&
		    (dai_link->platform_name || dai_link->platform_of_node)) {
			ret = snd_soc_dai_set_fmt(card->rtd[i].cpu_dai,
						  dai_fmt);
			if (ret != 0 && ret != -ENOTSUPP)
				dev_warn(card->rtd[i].cpu_dai->dev,
					 "ASoC: Failed to set DAI format: %d\n",
					 ret);
		} else if (dai_fmt) {
			/* Flip the polarity for the "CPU" end */
			dai_fmt &= ~SND_SOC_DAIFMT_MASTER_MASK;
			switch (dai_link->dai_fmt &
				SND_SOC_DAIFMT_MASTER_MASK) {
			case SND_SOC_DAIFMT_CBM_CFM:
				dai_fmt |= SND_SOC_DAIFMT_CBS_CFS;
				break;
			case SND_SOC_DAIFMT_CBM_CFS:
				dai_fmt |= SND_SOC_DAIFMT_CBS_CFM;
				break;
			case SND_SOC_DAIFMT_CBS_CFM:
				dai_fmt |= SND_SOC_DAIFMT_CBM_CFS;
				break;
			case SND_SOC_DAIFMT_CBS_CFS:
				dai_fmt |= SND_SOC_DAIFMT_CBM_CFM;
				break;
			}

			ret = snd_soc_dai_set_fmt(card->rtd[i].cpu_dai,
						  dai_fmt);
			if (ret != 0 && ret != -ENOTSUPP)
				dev_warn(card->rtd[i].cpu_dai->dev,
					 "ASoC: Failed to set DAI format: %d\n",
					 ret);
		}
	}

	snprintf(card->snd_card->shortname, sizeof(card->snd_card->shortname),
		 "%s", card->name);
	snprintf(card->snd_card->longname, sizeof(card->snd_card->longname),
		 "%s", card->long_name ? card->long_name : card->name);
	snprintf(card->snd_card->driver, sizeof(card->snd_card->driver),
		 "%s", card->driver_name ? card->driver_name : card->name);
	for (i = 0; i < ARRAY_SIZE(card->snd_card->driver); i++) {
		switch (card->snd_card->driver[i]) {
		case '_':
		case '-':
		case '\0':
			break;
		default:
			if (!isalnum(card->snd_card->driver[i]))
				card->snd_card->driver[i] = '_';
			break;
		}
	}

	if (card->late_probe) {
		ret = card->late_probe(card);
		if (ret < 0) {
			dev_err(card->dev, "ASoC: %s late_probe() failed: %d\n",
				card->name, ret);
			goto probe_aux_dev_err;
		}
	}

	if (card->fully_routed)
		snd_soc_dapm_auto_nc_pins(card);

	snd_soc_dapm_new_widgets(card);

	ret = snd_card_register(card->snd_card);
	if (ret < 0) {
		dev_err(card->dev, "ASoC: failed to register soundcard %d\n",
				ret);
		goto probe_aux_dev_err;
	}

#ifdef CONFIG_SND_SOC_AC97_BUS
	/* register any AC97 codecs */
	for (i = 0; i < card->num_rtd; i++) {
		ret = soc_register_ac97_dai_link(&card->rtd[i]);
		if (ret < 0) {
			dev_err(card->dev,
				"ASoC: failed to register AC97: %d\n", ret);
			while (--i >= 0)
				soc_unregister_ac97_dai_link(&card->rtd[i]);
			goto probe_aux_dev_err;
		}
	}
#endif

	card->instantiated = 1;
	snd_soc_dapm_sync(&card->dapm);
	mutex_unlock(&card->mutex);

	return 0;

probe_aux_dev_err:
	for (i = 0; i < card->num_aux_devs; i++)
		soc_remove_aux_dev(card, i);

probe_dai_err:
	soc_remove_dai_links(card);

card_probe_error:
	if (card->remove)
		card->remove(card);

	snd_card_free(card->snd_card);

base_error:
	mutex_unlock(&card->mutex);

	return ret;
}

static int soc_bind_dai_link(struct snd_soc_card *card, int num)
{
	struct snd_soc_dai_link *dai_link = &card->dai_link[num];
	struct snd_soc_pcm_runtime *rtd = &card->rtd[num];
	struct snd_soc_dai_link_component *codecs = dai_link->codecs;
	struct snd_soc_dai_link_component cpu_dai_component;
	struct snd_soc_dai **codec_dais = rtd->codec_dais;
	struct snd_soc_platform *platform;
	const char *platform_name;
	int i;

	dev_dbg(card->dev, "ASoC: binding %s at idx %d\n", dai_link->name, num);

	cpu_dai_component.name = dai_link->cpu_name;
	cpu_dai_component.of_node = dai_link->cpu_of_node;
	cpu_dai_component.dai_name = dai_link->cpu_dai_name;
	// 匹配和machine匹配的cpu dai driver
	rtd->cpu_dai = snd_soc_find_dai(&cpu_dai_component);
	if (!rtd->cpu_dai) {
		dev_err(card->dev, "ASoC: CPU DAI %s not registered\n",
			dai_link->cpu_dai_name);
		return -EPROBE_DEFER;
	}

	rtd->num_codecs = dai_link->num_codecs;

	/* Find CODEC from registered CODECs */
	for (i = 0; i < rtd->num_codecs; i++) {
		codec_dais[i] = snd_soc_find_dai(&codecs[i]);
		if (!codec_dais[i]) {
			dev_err(card->dev, "ASoC: CODEC DAI %s not registered\n",
				codecs[i].dai_name);
			return -EPROBE_DEFER;
		}
	}

	/* Single codec links expect codec and codec_dai in runtime data */
	rtd->codec_dai = codec_dais[0];
	rtd->codec = rtd->codec_dai->codec;

	/* if there's no platform we match on the empty platform */
	platform_name = dai_link->platform_name;
	if (!platform_name && !dai_link->platform_of_node)
		platform_name = "snd-soc-dummy";

	/* find one from the set of registered platforms */
	list_for_each_entry(platform, &platform_list, list) {
		if (dai_link->platform_of_node) {
			// 根据设备数节点进行匹配
			if (platform->dev->of_node !=
			    dai_link->platform_of_node)
				continue;
		} else {
			// 根据name进行匹配
			if (strcmp(platform->component.name, platform_name))
				continue;
		}
		
		rtd->platform = platform;
	}
	if (!rtd->platform) {
		dev_err(card->dev, "ASoC: platform %s not registered\n",
			dai_link->platform_name);
		return -EPROBE_DEFER;
	}

	card->num_rtd++;

	return 0;
}
```