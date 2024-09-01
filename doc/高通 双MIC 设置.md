android O中设置双MIC降噪，需要在build.prop添加属性"ro.vendor.audio.sdk.fluencetype"属性。

属性值位于`hardware/qcom/audio/configs/msm8953/msm8953.mk`
```
##fluencetype can be "fluence" or "fluencepro" or "none"
PRODUCT_PROPERTY_OVERRIDES += \
ro.vendor.audio.sdk.fluencetype=fluence\
persist.vendor.audio.fluence.voicecall=true\
persist.vendor.audio.fluence.voicerec=false\
persist.vendor.audio.fluence.speaker=true
```

代码解读位置：

`hardware/qcom/audio/hal/audio_hw.c`

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
    ······
    property_get("ro.vendor.audio.sdk.fluencetype", my_data->fluence_cap, "");			//解读属性值
    if (!strncmp("fluencepro", my_data->fluence_cap, sizeof("fluencepro"))) {
        my_data->fluence_type = FLUENCE_QUAD_MIC | FLUENCE_DUAL_MIC;					// 4MIC或者双MIC
    } else if (!strncmp("fluence", my_data->fluence_cap, sizeof("fluence"))) {
        my_data->fluence_type = FLUENCE_DUAL_MIC;						// 双MIC
    } else {
        my_data->fluence_type = FLUENCE_NONE;							// 单MIC
    }

    if (my_data->fluence_type != FLUENCE_NONE) {
        property_get("persist.vendor.audio.fluence.voicecall",value,"");
        if (!strncmp("true", value, sizeof("true"))) {
            my_data->fluence_in_voice_call = true;
        }
        
        property_get("persist.vendor.audio.fluence.voicerec",value,"");
        if (!strncmp("true", value, sizeof("true"))) {
            my_data->fluence_in_voice_rec = true;
        }
        
        property_get("persist.vendor.audio.fluence.audiorec",value,"");
        if (!strncmp("true", value, sizeof("true"))) {
            my_data->fluence_in_audio_rec = true;
        }
        
        property_get("persist.vendor.audio.fluence.speaker",value,"");
        if (!strncmp("true", value, sizeof("true"))) {
            my_data->fluence_in_spkr_mode = true;
        }
		        property_get("persist.vendor.audio.fluence.mode",value,"");
        if (!strncmp("broadside", value, sizeof("broadside"))) {
            my_data->fluence_mode = FLUENCE_BROADSIDE;
        }

        property_get("persist.vendor.audio.fluence.hfpcall",value,"");
        if (!strncmp("true", value, sizeof("true"))) {
            my_data->fluence_in_hfp_call = true;
        }
    }
	······
}
```

Tony Liu

2018-2-2