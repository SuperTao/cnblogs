//添加MIC设置参数
```
/hal/audio_extn/audio_extn.c
@@ -75,6 +75,7 @@ struct audio_extn_module {
     bool ras_enabled;
     struct aptx_dec_bt_addr addr;
     struct audio_device *adev;
+    int mic_choose;
 };
 
 static struct audio_extn_module aextnmod;
@@ -838,6 +839,7 @@ void audio_extn_set_parameters(struct audio_device *adev,
    if (adev->offload_effects_set_parameters != NULL)
        adev->offload_effects_set_parameters(parms);
    audio_extn_set_aptx_dec_bt_addr(adev, parms);
+   audio_extn_set_mic_choose_parameters(parms);
 }
 
 void audio_extn_get_parameters(const struct audio_device *adev,
@@ -1478,3 +1480,29 @@ int audio_extn_set_device_cfg_params(struct audio_device *adev,
 
     return 0;
 }
// 获取mic参数
int audio_extn_get_mic_choose_parameters(void)
{
    ALOGD("%s: mic_choose:%d", __func__, aextnmod.mic_choose);
    return aextnmod.mic_choose;
}
// 设置mic参数
void audio_extn_set_mic_choose_parameters(struct str_parms *parms)
{
    int ret;
    char value[32] = {0};
    ret = str_parms_get_str(parms, "MIC_CHOOSE", value, sizeof(value));
    ALOGD("mic_choose_ret:%d", ret);
    if (ret >= 0) {
        if (strcmp(value, "primary_mic") == 0) {
            aextnmod.mic_choose = 1;
        }else if (strcmp(value, "secondary_mic") == 0) {
            aextnmod.mic_choose = 2;
        } else {
            aextnmod.mic_choose = 0;
        }
    } else {
        aextnmod.mic_choose = 0;
    }
    ALOGD("%s: mic_choose:%d, value:%s", __func__, aextnmod.mic_choose, value);
}
```

//头文件中声明

/hal/audio_extn/audio_extn.h
```
+
+
+int audio_extn_get_mic_choose_parameters(void);
+
+void audio_extn_set_mic_choose_parameters(struct str_parms *parms);
```


hal/msm8916/platform.c
```
@@ -541,6 +541,7 @@ static const char * const device_table[SND_DEVICE_MAX] = {
     [SND_DEVICE_IN_UNPROCESSED_THREE_MIC] = "three-mic",
     [SND_DEVICE_IN_UNPROCESSED_QUAD_MIC] = "quad-mic",
     [SND_DEVICE_IN_UNPROCESSED_HEADSET_MIC] = "headset-mic",
	 //添加mic参数，与mixer_paths_mtp.xml对应
+    [SND_DEVICE_IN_SECONDARY_MIC] = "secondary-mic",
 };
 
 // Platform specific backend bit width table
@@ -683,6 +684,7 @@ static int acdb_device_table[SND_DEVICE_MAX] = {
     [SND_DEVICE_IN_UNPROCESSED_THREE_MIC] = 145,
     [SND_DEVICE_IN_UNPROCESSED_QUAD_MIC] = 146,
     [SND_DEVICE_IN_UNPROCESSED_HEADSET_MIC] = 147,
	 //自定义ID
+    [SND_DEVICE_IN_SECONDARY_MIC] = 170,
 };
 
 struct name_to_index {
@@ -4313,6 +4315,20 @@ snd_device_t platform_get_input_snd_device(void *platform, audio_devices_t out_d
                     snd_device = SND_DEVICE_IN_HANDSET_DMIC;
                     platform_set_echo_reference(adev, true, out_device);
                 }
+
+                /*
+                * 上层通过 AudioManager.setParameters("MIC_CHOOSE=xxx") 选择使用主副麦
+                * 主麦：MIC_CHOOSE=primary_mic
+                * 副麦：MIC_CHOOSE=secondary_mic
+                */
+                int mic_choose = 0;
+                mic_choose = audio_extn_get_mic_choose_parameters();
+                if (mic_choose == 1) {
+                    snd_device = SND_DEVICE_IN_HANDSET_MIC;
+                } else if (mic_choose == 2) {
+                    snd_device = SND_DEVICE_IN_SECONDARY_MIC;
+                }
+                ALOGD("%s: snd_device mic_choose (%s)", __func__, device_table[snd_device]);
             }
         }
     } else if (source == AUDIO_SOURCE_FM_TUNER) {
```

hal/msm8916/platform.h

```
     SND_DEVICE_IN_UNPROCESSED_THREE_MIC,
     SND_DEVICE_IN_UNPROCESSED_QUAD_MIC,
     SND_DEVICE_IN_UNPROCESSED_HEADSET_MIC,
+    SND_DEVICE_IN_SECONDARY_MIC,
     SND_DEVICE_IN_END,
 
     SND_DEVICE_MAX = SND_DEVICE_IN_END,
```

//	 设置MIC寄存器参数。
```
/configs/msm8953/mixer_paths_mtp.xml

           <path name="wsa-speaker-and-headphones" />
     </path>
 
+    <path name="secondary-mic">
+       <path name="adc3"/>
+   </path>
 </mixer>

```