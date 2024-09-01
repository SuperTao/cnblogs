记录一下高通音频配置文件mixer_paths.xml初始化过程。参考代码基于Android O。

hardware/qcom/audio/hal/audio_hw.c
```
struct audio_module HAL_MODULE_INFO_SYM = {
    .common = {
        .tag = HARDWARE_MODULE_TAG,
        .module_api_version = AUDIO_MODULE_API_VERSION_0_1,
        .hal_api_version = HARDWARE_HAL_API_VERSION,
        .id = AUDIO_HARDWARE_MODULE_ID,
        .name = "QCOM Audio HAL",
        .author = "The Linux Foundation",
        .methods = &hal_module_methods,
    },   
};


对应操作方法
static struct hw_module_methods_t hal_module_methods = {
    .open = adev_open,
};


static int adev_open(const hw_module_t *module, const char *name,
                     hw_device_t **device)
{
	......
    adev->platform = platform_init(adev);			// 平台初始化函数
    if (!adev->platform) {
        free(adev->snd_dev_ref_cnt);
        free(adev);
        ALOGE("%s: Failed to init platform data, aborting.", __func__);
        *device = NULL;
        pthread_mutex_unlock(&adev_init_lock);
        pthread_mutex_destroy(&adev->lock);
        return -EINVAL;
    }
	......
}
```

根据不同的芯片找到对应的文件
hardware/qcom/audio/hal/msm8916/platform.c
```
void *platform_init(struct audio_device *adev)
{
    char value[PROPERTY_VALUE_MAX];
    struct platform_data *my_data = NULL;
    int snd_card_num = 0;
    const char *snd_card_name;
    char mixer_xml_path[MAX_MIXER_XML_PATH],ffspEnable[PROPERTY_VALUE_MAX];
    const char *mixer_ctl_name = "Set HPX ActiveBe";
    struct mixer_ctl *ctl = NULL;
    int idx;
    int wsaCount =0;
    bool is_wsa_combo_supported = false;

    snd_card_num = audio_extn_utils_get_snd_card_num();     //获取声卡的的编号
    if(snd_card_num < 0) {
        ALOGE("%s: Unable to find correct sound card", __func__);
        return NULL;
    }

    adev->snd_card = snd_card_num;
    ALOGD("%s: Opened sound card:%d", __func__, snd_card_num);

    adev->mixer = mixer_open(snd_card_num);
	    if (!adev->mixer) {
        ALOGE("%s: Unable to open the mixer card: %d", __func__,
               snd_card_num);
        return NULL;
    }

    snd_card_name = mixer_get_name(adev->mixer);
    ALOGD("%s: snd_card_name: %s", __func__, snd_card_name);

    my_data = calloc(1, sizeof(struct platform_data));

    if (!my_data) {
        ALOGE("failed to allocate platform data");
        return NULL;
    }

    my_data->hw_info = hw_info_init(snd_card_name);     //初始化hw_info结构体
    if (!my_data->hw_info) {
        ALOGE("%s: Failed to init hardware info", __func__);
        free(my_data);
        return NULL;
    }

    query_platform(snd_card_name, mixer_xml_path);                              //query_platform,根据设备名称找到xml文件的路径
    ALOGD("%s: mixer path file is %s", __func__,
                            mixer_xml_path);
    if (audio_extn_read_xml(adev, snd_card_num, mixer_xml_path,
                            MIXER_XML_PATH_AUXPCM) == -ENOSYS) {
        adev->audio_route = audio_route_init(snd_card_num,					// 解析xml文件
                                         mixer_xml_path);
    }
    if (!adev->audio_route) {
        ALOGE("%s: Failed to init audio route controls, aborting.",
               __func__);
			           free(my_data);
        mixer_close(adev->mixer);
        return NULL;
    }
    update_codec_type(snd_card_name);
    update_interface(snd_card_name);
    ......
 
}
```

query_platform定义如下
```
static void query_platform(const char *snd_card_name,
                                      char *mixer_xml_path)
{
    if (!strncmp(snd_card_name, "msm8x16-snd-card-mtp",
                 sizeof("msm8x16-snd-card-mtp"))) {
        strlcpy(mixer_xml_path, MIXER_XML_PATH_MTP,
                sizeof(MIXER_XML_PATH_MTP));

        msm_device_to_be_id = msm_device_to_be_id_internal_codec;
        msm_be_id_array_len  =
            sizeof(msm_device_to_be_id_internal_codec) / sizeof(msm_device_to_be_id_internal_codec[0]);

		......
				
	} else if (!strncmp(snd_card_name, "msm8953-snd-card-mtp",                            // 根据设备名称使用不同的xml文件
                 sizeof("msm8953-snd-card-mtp"))) {
        strlcpy(mixer_xml_path, MIXER_XML_PATH_MTP,
                sizeof(MIXER_XML_PATH_MTP));
        msm_device_to_be_id = msm_device_to_be_id_internal_codec;
        msm_be_id_array_len  =
            sizeof(msm_device_to_be_id_internal_codec) / sizeof(msm_device_to_be_id_internal_codec[0]);
		
		......
}
```

设备名称通过可以在adb中查看`cat /proc/asound/cards`，如下：		
```
Tony:/ # cat /proc/asound/cards
cat /proc/asound/cards
 0 [msm8953sndcardm]: msm8953-snd-car - msm8953-snd-card-mtp
                      msm8953-snd-card-mtp	
```

因此可以知道选取的文件
`#define MIXER_XML_PATH_MTP "/etc/mixer_paths_mtp.xml"`

xml文件解析函数位于`system/media/audio_route/audio_route.c`
```
struct audio_route *audio_route_init(unsigned int card, const char *xml_path)
{
    struct config_parse_state state;
    XML_Parser parser;
    FILE *file;
    int bytes_read;
    void *buf;
    struct audio_route *ar;

    ar = calloc(1, sizeof(struct audio_route));
    if (!ar)
        goto err_calloc;

    ar->mixer = mixer_open(card);
    if (!ar->mixer) {
        ALOGE("Unable to open the mixer, aborting.");
        goto err_mixer_open;
    }   

    ar->mixer_path = NULL;
    ar->mixer_path_size = 0;
    ar->num_mixer_paths = 0;

    /* allocate space for and read current mixer settings */
    if (alloc_mixer_state(ar) < 0)
        goto err_mixer_state;

    /* use the default XML path if none is provided */
    if (xml_path == NULL)
        xml_path = MIXER_XML_PATH;

    file = fopen(xml_path, "r");

    if (!file) {
        ALOGE("Failed to open %s: %s", xml_path, strerror(errno));
		    parser = XML_ParserCreate(NULL);
    if (!parser) {
        ALOGE("Failed to create XML parser");
        goto err_parser_create;
    }

    memset(&state, 0, sizeof(state));
    state.ar = ar;
    XML_SetUserData(parser, &state);
    XML_SetElementHandler(parser, start_tag, end_tag);

    for (;;) {
        buf = XML_GetBuffer(parser, BUF_SIZE);
        if (buf == NULL)
            goto err_parse;

        bytes_read = fread(buf, 1, BUF_SIZE, file);
        if (bytes_read < 0)
            goto err_parse;

        if (XML_ParseBuffer(parser, bytes_read,
                            bytes_read == 0) == XML_STATUS_ERROR) {
            ALOGE("Error in mixer xml (%s)", MIXER_XML_PATH);
            goto err_parse;
        }

        if (bytes_read == 0)
            break;
    }

    /* apply the initial mixer values, and save them so we can reset the
       mixer to the original values */
    audio_route_update_mixer(ar);
    save_mixer_state(ar);

    XML_ParserFree(parser);
	    fclose(file);
    return ar;
    ......
}
```

xml位于源码位置hardware/qcom/audio/configs/msm8953/

xml文件存放的路径更改了,android O的路径位于/vendor/etc/下面，android N位于/system/etc/

mixer_paths_mtp.xml分析，可参考http://bbs.gfan.com/android-7626545-1-1.html

文件的开头会写一些默认的参数，也就是<ctl>里面的内容，就是默认的参数。

然后不同的设备，也就是不同的<path>标签里面的内容，<path>标签中的值表示不同的设备。

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<mixer>
    <!-- These are the initial mixer settings -->
    <ctl name="Voice Rx Device Mute" id="0" value="0" />
    <ctl name="Voice Rx Device Mute" id="1" value="-1" />
    <ctl name="Voice Rx Device Mute" id="2" value="20" />
    <ctl name="Voice Tx Mute" id="0" value="0" />
    <ctl name="Voice Tx Mute" id="1" value="-1" />
    <ctl name="Voice Tx Mute" id="2" value="500" />
    <ctl name="Voice Rx Gain" id="0" value="0" />
    <ctl name="Voice Rx Gain" id="1" value="-1" />
    <ctl name="Voice Rx Gain" id="2" value="20" />
	......
	    <path name="handset">						<!-- 电话听筒 -->
        <ctl name="RX1 MIX1 INP1" value="RX1" />
        <ctl name="RDAC2 MUX" value="RX1" />
        <ctl name="RX1 Digital Volume" value="84" />
        <ctl name="EAR PA Gain" value="POS_6_DB" />
        <ctl name="EAR_S" value="Switch" />
    </path>

    <path name="handset-mic">						
        <path name="adc1" />
        <ctl name="IIR1 INP1 MUX" value="DEC1" />
    </path>

    <path name="headphones">						<!-- 耳机 -->
        <ctl name="MI2S_RX Channels" value="Two" />
        <ctl name="RX1 MIX1 INP1" value="RX1" />
        <ctl name="RX2 MIX1 INP1" value="RX2" />
        <ctl name="RX HPH Mode" value="HD2" />
        <ctl name="COMP0 RX1" value="1" />
        <ctl name="COMP0 RX2" value="1" />
        <ctl name="RDAC2 MUX" value="RX2" />
        <ctl name="HPHL" value="Switch" />
        <ctl name="HPHR" value="Switch" />
    </path>

    <path name="headset-mic">
        <path name="adc2" />
        <ctl name="ADC2 MUX" value="INP2" />
        <ctl name="IIR1 INP1 MUX" value="DEC1" />
    </path>
	......

```

Tony Liu

2018-1-29