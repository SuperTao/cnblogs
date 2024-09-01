AudioTrack最终会调用AudioPolicyManager::getOutput();
```
frameworks\av\services\audiopolicy\managerdefault\AudioPolicyManager.cpp
audio_io_handle_t AudioPolicyManager::getOutput(audio_stream_type_t stream,
                                                uint32_t samplingRate,
                                                audio_format_t format,
                                                audio_channel_mask_t channelMask,
                                                audio_output_flags_t flags,
                                                const audio_offload_info_t *offloadInfo)
{
    //APM_AudioPolicyManager: getOutput() device 2, stream 1, samplingRate 0, format 0, channelMask 3, flags 0
    // stream = 1, AUDIO_STREAM_SYSTEM, 得到的strategy = STRATEGY_MEDIA
    routing_strategy strategy = getStrategy(stream);
    // AUDIO_DEVICE_NONE & AUDIO_DEVICE_OUT_SPEAKER
    audio_devices_t device = getDeviceForStrategy(strategy, false /*fromCache*/);
    //APM_AudioPolicyManager: getOutput() device 2, stream 1, samplingRate 0, format 0, channelMask 3, flags 0
    ALOGV("getOutput() device %d, stream %d, samplingRate %d, format %x, channelMask %x, flags %x",
          device, stream, samplingRate, format, channelMask, flags);
    ALOGE("Liutao getOutput() device %d, stream %d, samplingRate %d, format %x, channelMask %x, flags %x",
          device, stream, samplingRate, format, channelMask, flags);

    return getOutputForDevice(device, AUDIO_SESSION_ALLOCATE, stream, samplingRate, format,
                              channelMask, flags, offloadInfo);
}
```

#### 根据stream获取strategy

StreamType指PCM的生成类型。是播放电影产生的？还是通话产生的？

STRATEGY指针对某一中stream,该采用的策略。在策略里面，会根据其他信息来具体选定某一个具体的Audio Devices

```
routing_strategy AudioPolicyManager::getStrategy(audio_stream_type_t stream) const
{
    ALOG_ASSERT(stream != AUDIO_STREAM_PATCH,"getStrategy() called for AUDIO_STREAM_PATCH");
    return mEngine->getStrategyForStream(stream);
}

// 不同的stream类型对应的stragegy，播放音乐，系统铃声，stream = 1, AUDIO_STREAM_SYSTEM = 1,
frameworks\av\services\audiopolicy\enginedefault\src\Engine.cpp
routing_strategy Engine::getStrategyForStream(audio_stream_type_t stream)
{
    // stream to strategy mapping
    switch (stream) {
    case AUDIO_STREAM_VOICE_CALL:
    case AUDIO_STREAM_BLUETOOTH_SCO:
        return STRATEGY_PHONE;
    case AUDIO_STREAM_RING:
    case AUDIO_STREAM_ALARM:
        return STRATEGY_SONIFICATION;
    case AUDIO_STREAM_NOTIFICATION:
        return STRATEGY_SONIFICATION_RESPECTFUL;
    case AUDIO_STREAM_DTMF:
        return STRATEGY_DTMF;
    default:
        ALOGE("unknown stream type %d", stream);
    case AUDIO_STREAM_SYSTEM:
        // NOTE: SYSTEM stream uses MEDIA strategy because muting music and switching outputs
        // while key clicks are played produces a poor result
    case AUDIO_STREAM_MUSIC:
        return STRATEGY_MEDIA;
    case AUDIO_STREAM_ENFORCED_AUDIBLE:
        return STRATEGY_ENFORCED_AUDIBLE;
    case AUDIO_STREAM_TTS:
        return STRATEGY_TRANSMITTED_THROUGH_SPEAKER;
    case AUDIO_STREAM_ACCESSIBILITY:
        return STRATEGY_ACCESSIBILITY;
    case AUDIO_STREAM_REROUTING:
        return STRATEGY_REROUTING;
    }
}

frameworks\av\services\audiopolicy\common\include\RoutingStrategy.h
enum routing_strategy {
    STRATEGY_MEDIA,
    STRATEGY_PHONE,
    STRATEGY_SONIFICATION,
    STRATEGY_SONIFICATION_RESPECTFUL,
    STRATEGY_DTMF,
    STRATEGY_ENFORCED_AUDIBLE,
    STRATEGY_TRANSMITTED_THROUGH_SPEAKER,
    STRATEGY_ACCESSIBILITY,
    STRATEGY_REROUTING,
    NUM_STRATEGIES
};

// STREAM类型定义
typedef enum {
    AUDIO_STREAM_DEFAULT = -1, // (-1)
    AUDIO_STREAM_MIN = 0,
    AUDIO_STREAM_VOICE_CALL = 0,
    AUDIO_STREAM_SYSTEM = 1,
    AUDIO_STREAM_RING = 2,
    AUDIO_STREAM_MUSIC = 3,
    AUDIO_STREAM_ALARM = 4,
    AUDIO_STREAM_NOTIFICATION = 5,
    AUDIO_STREAM_BLUETOOTH_SCO = 6,
    AUDIO_STREAM_ENFORCED_AUDIBLE = 7,
    AUDIO_STREAM_DTMF = 8,
    AUDIO_STREAM_TTS = 9,
    AUDIO_STREAM_ACCESSIBILITY = 10,
    AUDIO_STREAM_REROUTING = 11,
    AUDIO_STREAM_PATCH = 12,
    AUDIO_STREAM_PUBLIC_CNT = 11, // (ACCESSIBILITY + 1)
    AUDIO_STREAM_FOR_POLICY_CNT = 12, // PATCH
    AUDIO_STREAM_CNT = 13, // (PATCH + 1)
} audio_stream_type_t;
```

#### 根据Strategy获取device
```
audio_devices_t AudioPolicyManager::getDeviceForStrategy(routing_strategy strategy,
                                                         bool fromCache)
{
    // Routing
    // see if we have an explicit route
    // scan the whole RouteMap, for each entry, convert the stream type to a strategy
    // (getStrategy(stream)).
    // if the strategy from the stream type in the RouteMap is the same as the argument above,
    // and activity count is non-zero and the device in the route descriptor is available
    // then select this device.
    for (size_t routeIndex = 0; routeIndex < mOutputRoutes.size(); routeIndex++) {
        sp<SessionRoute> route = mOutputRoutes.valueAt(routeIndex);
        routing_strategy routeStrategy = getStrategy(route->mStreamType);
        // 判断输入的STRAGEGY是否在系统支持的mOutputRoutes列表里面
        if ((routeStrategy == strategy) && route->isActive() &&
                (mAvailableOutputDevices.indexOf(route->mDeviceDescriptor) >= 0)) {
            return route->mDeviceDescriptor->type();
        }
    }

    if (fromCache) {
        ALOGVV("getDeviceForStrategy() from cache strategy %d, device %x",
              strategy, mDeviceForStrategy[strategy]);
        return mDeviceForStrategy[strategy];
    }
    // 传入的stragegy = STRATEGY_MEDIA
    return mEngine->getDeviceForStrategy(strategy);
}

audio_devices_t Engine::getDeviceForStrategy(routing_strategy strategy) const
{
    DeviceVector availableOutputDevices = mApmObserver->getAvailableOutputDevices();
    DeviceVector availableInputDevices = mApmObserver->getAvailableInputDevices();

    const SwAudioOutputCollection &outputs = mApmObserver->getOutputs();

    return getDeviceForStrategyInt(strategy, availableOutputDevices,
                                   availableInputDevices, outputs);
}

frameworks\av\services\audiopolicy\enginedefault\src\Engine.cpp
audio_devices_t Engine::getDeviceForStrategyInt(routing_strategy strategy,
                                                DeviceVector availableOutputDevices,
                                                DeviceVector availableInputDevices,
                                                const SwAudioOutputCollection &outputs) const
{
    uint32_t device = AUDIO_DEVICE_NONE;
    uint32_t availableOutputDevicesType = availableOutputDevices.types();
    bool isFmA2dpConcurrencyOn = property_get_bool("vendor.fm.a2dp.conc.disabled", false);

    // Do not support a2dp device when FM is active based on concurrency property
    if (isFmA2dpConcurrencyOn && (availableOutputDevicesType & AUDIO_DEVICE_OUT_FM)) {
        ALOGV("FM a2dp concurrency is set, not considering a2dp for device selection");
        availableOutputDevicesType = availableOutputDevicesType & ~AUDIO_DEVICE_OUT_ALL_A2DP;
    }

// 传入的stragegy = STRATEGY_MEDIA = 0
    switch (strategy) {
    // FIXME: STRATEGY_REROUTING follow STRATEGY_MEDIA for now
    case STRATEGY_REROUTING:
    case STRATEGY_MEDIA: {
        uint32_t device2 = AUDIO_DEVICE_NONE;
        // 是否在打电话
        if (isInCall() && (device == AUDIO_DEVICE_NONE)) {
            // when in call, get the device for Phone strategy
            device = getDeviceForStrategy(STRATEGY_PHONE);
            break;
        }

        if (strategy != STRATEGY_SONIFICATION) {
            // no sonification on remote submix (e.g. WFD)
            // 查看是否有远程设备
            if (availableOutputDevices.getDevice(AUDIO_DEVICE_OUT_REMOTE_SUBMIX,
                                                 String8("0")) != 0) {
                device2 = availableOutputDevices.types() & AUDIO_DEVICE_OUT_REMOTE_SUBMIX;
            }
        }
        // 打电话, 并且播放音乐还有系统铃声时的处理
        if (isInCall() && (strategy == STRATEGY_MEDIA)) {
            device = getDeviceForStrategyInt(
                    STRATEGY_PHONE, availableOutputDevices, availableInputDevices, outputs);
            break;
        }
        // 蓝牙耳机
        if ((device2 == AUDIO_DEVICE_NONE) &&
                (mForceUse[AUDIO_POLICY_FORCE_FOR_MEDIA] != AUDIO_POLICY_FORCE_NO_BT_A2DP) &&
                (outputs.isA2dpOnPrimary() || (outputs.getA2dpOutput() != 0))) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_BLUETOOTH_A2DP;
            if (device2 == AUDIO_DEVICE_NONE) {
                device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_BLUETOOTH_A2DP_HEADPHONES;
            }
            if (device2 == AUDIO_DEVICE_NONE) {
                device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_BLUETOOTH_A2DP_SPEAKER;
            }
        }
        // SPEAKER
        if ((device2 == AUDIO_DEVICE_NONE) &&
            (mForceUse[AUDIO_POLICY_FORCE_FOR_MEDIA] == AUDIO_POLICY_FORCE_SPEAKER)) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_SPEAKER;
        }
        // 听筒
        if (device2 == AUDIO_DEVICE_NONE) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_WIRED_HEADPHONE;
        }
        if (device2 == AUDIO_DEVICE_NONE) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_LINE;
        }
        // 有线耳机
        if (device2 == AUDIO_DEVICE_NONE) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_WIRED_HEADSET;
        }
        // USB耳机
        if (device2 == AUDIO_DEVICE_NONE) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_USB_HEADSET;
        }
        if (device2 == AUDIO_DEVICE_NONE) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_USB_ACCESSORY;
        }
        if (device2 == AUDIO_DEVICE_NONE) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_USB_DEVICE;
        }
        if (device2 == AUDIO_DEVICE_NONE) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_DGTL_DOCK_HEADSET;
        }
        if ((device2 == AUDIO_DEVICE_NONE) && (strategy != STRATEGY_SONIFICATION) &&
                (device == AUDIO_DEVICE_NONE)) {
            // no sonification on aux digital (e.g. HDMI)
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_AUX_DIGITAL;
        }
        if ((device2 == AUDIO_DEVICE_NONE) && (strategy != STRATEGY_SONIFICATION) &&
                (mForceUse[AUDIO_POLICY_FORCE_FOR_DOCK] == AUDIO_POLICY_FORCE_ANALOG_DOCK)) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_ANLG_DOCK_HEADSET;
        }
        if ((device2 == AUDIO_DEVICE_NONE) && (strategy != STRATEGY_SONIFICATION) &&
                (device == AUDIO_DEVICE_NONE)) {
            // no sonification on WFD sink
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_PROXY;
        }
        if (device2 == AUDIO_DEVICE_NONE) {
            device2 = availableOutputDevicesType & AUDIO_DEVICE_OUT_SPEAKER;
        }
        int device3 = AUDIO_DEVICE_NONE;
        // 几个设备同时输出
        if (strategy == STRATEGY_MEDIA) {
            // ARC, SPDIF and AUX_LINE can co-exist with others.
            device3 = availableOutputDevicesType & AUDIO_DEVICE_OUT_HDMI_ARC;
            device3 |= (availableOutputDevicesType & AUDIO_DEVICE_OUT_SPDIF);
            device3 |= (availableOutputDevicesType & AUDIO_DEVICE_OUT_AUX_LINE);
        }

        device2 |= device3;
        // device is DEVICE_OUT_SPEAKER if we come from case STRATEGY_SONIFICATION or
        // STRATEGY_ENFORCED_AUDIBLE, AUDIO_DEVICE_NONE otherwise
        device |= device2;

        // If hdmi system audio mode is on, remove speaker out of output list.
        if ((strategy == STRATEGY_MEDIA) &&
            (mForceUse[AUDIO_POLICY_FORCE_FOR_HDMI_SYSTEM_AUDIO] ==
                AUDIO_POLICY_FORCE_HDMI_SYSTEM_AUDIO_ENFORCED)) {
            device &= ~AUDIO_DEVICE_OUT_SPEAKER;
        }

        // for STRATEGY_SONIFICATION:
        // if SPEAKER was selected, and SPEAKER_SAFE is available, use SPEAKER_SAFE instead
        if ((strategy == STRATEGY_SONIFICATION) &&
                (device & AUDIO_DEVICE_OUT_SPEAKER) &&
                (availableOutputDevicesType & AUDIO_DEVICE_OUT_SPEAKER_SAFE)) {
            device |= AUDIO_DEVICE_OUT_SPEAKER_SAFE;
            device &= ~AUDIO_DEVICE_OUT_SPEAKER;
        }
        } break;

    default:
        ALOGW("getDeviceForStrategy() unknown strategy: %d", strategy);
        break;
    }

    if (device == AUDIO_DEVICE_NONE) {
        ALOGV("getDeviceForStrategy() no device found for strategy %d", strategy);
        device = mApmObserver->getDefaultOutputDevice()->type();
        ALOGE_IF(device == AUDIO_DEVICE_NONE,
                 "getDeviceForStrategy() no default device defined");
    }
    ALOGVV("getDeviceForStrategy() strategy %d, device %x", strategy, device);
    return device;
}
```

#### 为设备选择的输出通道

// APM_AudioPolicyManager: getOutput() device 2, stream 1, samplingRate 0, format 0, channelMask 3, flags 0

```
audio_io_handle_t AudioPolicyManager::getOutputForDevice(
        audio_devices_t device,
        audio_session_t session,
        audio_stream_type_t stream,
        uint32_t samplingRate,
        audio_format_t format,
        audio_channel_mask_t channelMask,
        audio_output_flags_t flags,
        const audio_offload_info_t *offloadInfo)

hardware\qcom\audio\hal\msm8916\platform.c
static int msm_device_to_be_id_internal_codec [][NO_COLS] = {
       {AUDIO_DEVICE_OUT_EARPIECE                       ,       34},
       {AUDIO_DEVICE_OUT_SPEAKER                        ,       34},
       {AUDIO_DEVICE_OUT_WIRED_HEADSET                  ,       34},
       {AUDIO_DEVICE_OUT_WIRED_HEADPHONE                ,       34},
       {AUDIO_DEVICE_OUT_BLUETOOTH_SCO                  ,       11},
       {AUDIO_DEVICE_OUT_BLUETOOTH_SCO_HEADSET          ,       11},
       {AUDIO_DEVICE_OUT_BLUETOOTH_SCO_CARKIT           ,       11},
       {AUDIO_DEVICE_OUT_BLUETOOTH_A2DP                 ,       -1},
       {AUDIO_DEVICE_OUT_BLUETOOTH_A2DP_HEADPHONES      ,       -1},
       {AUDIO_DEVICE_OUT_BLUETOOTH_A2DP_SPEAKER         ,       -1},
       {AUDIO_DEVICE_OUT_AUX_DIGITAL                    ,       4},
       {AUDIO_DEVICE_OUT_ANLG_DOCK_HEADSET              ,       9},
       {AUDIO_DEVICE_OUT_DGTL_DOCK_HEADSET              ,       9},
       {AUDIO_DEVICE_OUT_USB_ACCESSORY                  ,       -1},
       {AUDIO_DEVICE_OUT_USB_DEVICE                     ,       -1},
       {AUDIO_DEVICE_OUT_USB_HEADSET                    ,       -1},
       {AUDIO_DEVICE_OUT_REMOTE_SUBMIX                  ,       9},
       {AUDIO_DEVICE_OUT_PROXY                          ,       9},
       {AUDIO_DEVICE_OUT_FM                             ,       7},
       {AUDIO_DEVICE_OUT_FM_TX                          ,       8},
       {AUDIO_DEVICE_OUT_ALL                            ,      -1},
       {AUDIO_DEVICE_NONE                               ,      -1},
       {AUDIO_DEVICE_OUT_DEFAULT                        ,      -1},
};

static int msm_device_to_be_id_external_codec [][NO_COLS] = {
       {AUDIO_DEVICE_OUT_EARPIECE                       ,       2},
       {AUDIO_DEVICE_OUT_SPEAKER                        ,       2},
       {AUDIO_DEVICE_OUT_WIRED_HEADSET                  ,       41},
       {AUDIO_DEVICE_OUT_WIRED_HEADPHONE                ,       41},
       {AUDIO_DEVICE_OUT_BLUETOOTH_SCO                  ,       11},
       {AUDIO_DEVICE_OUT_BLUETOOTH_SCO_HEADSET          ,       11},
       {AUDIO_DEVICE_OUT_BLUETOOTH_SCO_CARKIT           ,       11},
       {AUDIO_DEVICE_OUT_BLUETOOTH_A2DP                 ,       -1},
       {AUDIO_DEVICE_OUT_BLUETOOTH_A2DP_HEADPHONES      ,       -1},
       {AUDIO_DEVICE_OUT_BLUETOOTH_A2DP_SPEAKER         ,       -1},
       {AUDIO_DEVICE_OUT_AUX_DIGITAL                    ,       4},
       {AUDIO_DEVICE_OUT_ANLG_DOCK_HEADSET              ,       9},
       {AUDIO_DEVICE_OUT_DGTL_DOCK_HEADSET              ,       9},
       {AUDIO_DEVICE_OUT_USB_ACCESSORY                  ,       -1},
       {AUDIO_DEVICE_OUT_USB_DEVICE                     ,       -1},
       {AUDIO_DEVICE_OUT_USB_HEADSET                    ,       -1},
       {AUDIO_DEVICE_OUT_REMOTE_SUBMIX                  ,       9},
       {AUDIO_DEVICE_OUT_PROXY                          ,       9},
       {AUDIO_DEVICE_OUT_FM                             ,       7},
       {AUDIO_DEVICE_OUT_FM_TX                          ,       8},
       {AUDIO_DEVICE_OUT_ALL                            ,      -1},
       {AUDIO_DEVICE_NONE                               ,      -1},
       {AUDIO_DEVICE_OUT_DEFAULT                        ,      -1},
};
```