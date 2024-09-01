记录一下AudioPolicy初始化过程。

```
frameworks\av\media\audioserver\audioserver.rc
service audioserver /system/bin/audioserver
    class main
    user audioserver
    # media gid needed for /dev/fm (radio) and for /data/misc/media (tee)
    group audio camera drmrpc inet media mediadrm net_bt net_bt_admin net_bw_acct oem_2901
    ioprio rt 4
    writepid /dev/cpuset/foreground/tasks /dev/stune/foreground/tasks
    onrestart restart audio-hal-2-0
	
frameworks\av\media\audioserver\main_audioserver.cpp
int main(int argc __unused, char **argv)     
		AudioFlinger::instantiate();
        AudioPolicyService::instantiate();
		

frameworks\av\services\audiopolicy\service\AudioPolicyService.cpp
void AudioPolicyService::onFirstRef()
{
    {
        Mutex::Autolock _l(mLock);

        // start tone playback thread
        mTonePlaybackThread = new AudioCommandThread(String8("ApmTone"), this);
        // start audio commands thread
        mAudioCommandThread = new AudioCommandThread(String8("ApmAudio"), this);
        // start output activity command thread
        mOutputCommandThread = new AudioCommandThread(String8("ApmOutput"), this);

        mAudioPolicyClient = new AudioPolicyClient(this);
        mAudioPolicyManager = createAudioPolicyManager(mAudioPolicyClient);				// 创建AudioPolicyManager
    }
    // load audio processing modules
    sp<AudioPolicyEffects>audioPolicyEffects = new AudioPolicyEffects();				// 加载声音效果文件
    {
        Mutex::Autolock _l(mLock);
        mAudioPolicyEffects = audioPolicyEffects;
    }
}

// 加载声音效果
AudioPolicyEffects::AudioPolicyEffects()
{
    status_t loadResult = loadAudioEffectXmlConfig();
    if (loadResult < 0) {
        ALOGW("Failed to load XML effect configuration, fallback to .conf");
        // load automatic audio effect modules
        if (access(AUDIO_EFFECT_VENDOR_CONFIG_FILE, R_OK) == 0) {
            loadAudioEffectConfig(AUDIO_EFFECT_VENDOR_CONFIG_FILE);
        } else if (access(AUDIO_EFFECT_DEFAULT_CONFIG_FILE, R_OK) == 0) {
            loadAudioEffectConfig(AUDIO_EFFECT_DEFAULT_CONFIG_FILE);
        }
    } else if (loadResult > 0) {
        ALOGE("Effect config is partially invalid, skipped %d elements", loadResult);
    }
}

system\media\audio\include\system\audio_effects\audio_effects_conf.h
#define AUDIO_EFFECT_DEFAULT_CONFIG_FILE "/system/etc/audio_effects.conf"
#define AUDIO_EFFECT_VENDOR_CONFIG_FILE "/vendor/etc/audio_effects.conf"

frameworks\av\services\audiopolicy\manager\AudioPolicyFactory.cpp
extern "C" AudioPolicyInterface* createAudioPolicyManager(
        AudioPolicyClientInterface *clientInterface)
{
    return new AudioPolicyManager(clientInterface);
}



AudioPolicyManager创建：

初始化主要读取xml文件，获取可以支持的设备类型，采样率等参数。
frameworks\av\services\audiopolicy\managerdefault\AudioPolicyManager.cpp

AudioPolicyManager::AudioPolicyManager(AudioPolicyClientInterface *clientInterface)
    :
#ifdef AUDIO_POLICY_TEST
    Thread(false),
#endif //AUDIO_POLICY_TEST
    mLimitRingtoneVolume(false), mLastVoiceVolume(-1.0f),
    mA2dpSuspended(false),
    mAudioPortGeneration(1),
    mBeaconMuteRefCount(0),
    mBeaconPlayingRefCount(0),
    mBeaconMuted(false),
    mTtsOutputAvailable(false),
    mMasterMono(false),
    mMusicEffectOutput(AUDIO_IO_HANDLE_NONE),
    mHasComputedSoundTriggerSupportsConcurrentCapture(false)
{
    mUidCached = getuid();
    mpClientInterface = clientInterface;

    // TODO: remove when legacy conf file is removed. true on devices that use DRC on the
    // DEVICE_CATEGORY_SPEAKER path to boost soft sounds, used to adjust volume curves accordingly.
    // Note: remove also speaker_drc_enabled from global configuration of XML config file.
    bool speakerDrcEnabled = false;

	
//hardware\qcom\audio\configs\msm8953\msm8953.mk
//USE_XML_AUDIO_POLICY_CONF := 1	
#ifdef USE_XML_AUDIO_POLICY_CONF
    // 设置声音曲线
    mVolumeCurves = new VolumeCurvesCollection();
    AudioPolicyConfig config(mHwModules, mAvailableOutputDevices, mAvailableInputDevices,
                             mDefaultOutputDevice, speakerDrcEnabled,
                             static_cast<VolumeCurvesCollection *>(mVolumeCurves));
    // 解析文件audio_policy_configuration.xml
    if (deserializeAudioPolicyXmlConfig(config) != NO_ERROR) {
#else
    mVolumeCurves = new StreamDescriptorCollection();
    AudioPolicyConfig config(mHwModules, mAvailableOutputDevices, mAvailableInputDevices,
                             mDefaultOutputDevice, speakerDrcEnabled);
    // #define AUDIO_POLICY_CONFIG_FILE "/system/etc/audio_policy.conf"
    // #define AUDIO_POLICY_VENDOR_CONFIG_FILE "/vendor/etc/audio_policy.conf"
    if ((ConfigParsingUtils::loadConfig(AUDIO_POLICY_VENDOR_CONFIG_FILE, config) != NO_ERROR) &&
            (ConfigParsingUtils::loadConfig(AUDIO_POLICY_CONFIG_FILE, config) != NO_ERROR)) {
#endif
        ALOGE("could not load audio policy configuration file, setting defaults");
        config.setDefault();
    }
    // must be done after reading the policy (since conditionned by Speaker Drc Enabling)
    mVolumeCurves->initializeVolumeCurves(speakerDrcEnabled);

    // Once policy config has been parsed, retrieve an instance of the engine and initialize it.
    audio_policy::EngineInstance *engineInstance = audio_policy::EngineInstance::getInstance();
    if (!engineInstance) {
        ALOGE("%s:  Could not get an instance of policy engine", __FUNCTION__);
        return;
    }
    // Retrieve the Policy Manager Interface
    mEngine = engineInstance->queryInterface<AudioPolicyManagerInterface>();
    if (mEngine == NULL) {
        ALOGE("%s: Failed to get Policy Engine Interface", __FUNCTION__);
        return;
    }
    mEngine->setObserver(this);
    status_t status = mEngine->initCheck();
    (void) status;
    ALOG_ASSERT(status == NO_ERROR, "Policy engine not initialized(err=%d)", status);

    // mAvailableOutputDevices and mAvailableInputDevices now contain all attached devices
    // open all output streams needed to access attached devices
    audio_devices_t outputDeviceTypes = mAvailableOutputDevices.types();
    audio_devices_t inputDeviceTypes = mAvailableInputDevices.types() & ~AUDIO_DEVICE_BIT_IN;
    for (size_t i = 0; i < mHwModules.size(); i++) {
        mHwModules[i]->mHandle = mpClientInterface->loadHwModule(mHwModules[i]->getName());
        ALOGE("Liutao %s: mHwModules.name: %s", __FUNCTION__,    mHwModules[i]->getName());
        /*
        打印出的模块名        // 对应xml的mudule.
        AudioPolicyManager: mHwModules.name: primary
        AudioPolicyManager: mHwModules.name: a2dp
        AudioPolicyManager: mHwModules.name: usb
        AudioPolicyManager: mHwModules.name: r_submix
        */
        if (mHwModules[i]->mHandle == 0) {
            ALOGW("could not open HW module %s", mHwModules[i]->getName());
            continue;
        }
        // open all output streams needed to access attached devices
        // except for direct output streams that are only opened when they are actually
        // required by an app.
        // This also validates mAvailableOutputDevices list
    // 对应xml的profile
        for (size_t j = 0; j < mHwModules[i]->mOutputProfiles.size(); j++)
        {
            const sp<IOProfile> outProfile = mHwModules[i]->mOutputProfiles[j];
//            ALOGE("Liutao %s: mOutputProfile.name: %s", __FUNCTION__, mHwModules[i]->mOutputProfiles.getName());

            if (!outProfile->hasSupportedDevices()) {
                ALOGW("Output profile contains no device on module %s", mHwModules[i]->getName());
                continue;
            }
            if ((outProfile->getFlags() & AUDIO_OUTPUT_FLAG_TTS) != 0) {
                mTtsOutputAvailable = true;
            }

            if ((outProfile->getFlags() & AUDIO_OUTPUT_FLAG_DIRECT) != 0) {
                continue;
            }
            audio_devices_t profileType = outProfile->getSupportedDevicesType();
            if ((profileType & mDefaultOutputDevice->type()) != AUDIO_DEVICE_NONE) {
                profileType = mDefaultOutputDevice->type();
            } else {
                // chose first device present in profile's SupportedDevices also part of
                // outputDeviceTypes
                profileType = outProfile->getSupportedDeviceForType(outputDeviceTypes);
            }
            if ((profileType & outputDeviceTypes) == 0) {
                continue;
            }
            sp<SwAudioOutputDescriptor> outputDesc = new SwAudioOutputDescriptor(outProfile,
                                                                                 mpClientInterface);
            const DeviceVector &supportedDevices = outProfile->getSupportedDevices();
            const DeviceVector &devicesForType = supportedDevices.getDevicesFromType(profileType);
            String8 address = devicesForType.size() > 0 ? devicesForType.itemAt(0)->mAddress
                    : String8("");

            outputDesc->mDevice = profileType;
            audio_config_t config = AUDIO_CONFIG_INITIALIZER;
            config.sample_rate = outputDesc->mSamplingRate;
            config.channel_mask = outputDesc->mChannelMask;
            config.format = outputDesc->mFormat;
            audio_io_handle_t output = AUDIO_IO_HANDLE_NONE;
            status_t status = mpClientInterface->openOutput(outProfile->getModuleHandle(),
                                                            &output,
                                                            &config,
                                                            &outputDesc->mDevice,
                                                            address,
                                                            &outputDesc->mLatency,
                                                            outputDesc->mFlags);

            if (status != NO_ERROR) {
                ALOGW("Cannot open output stream for device %08x on hw module %s",
                      outputDesc->mDevice,
                      mHwModules[i]->getName());
            } else {
                outputDesc->mSamplingRate = config.sample_rate;
                outputDesc->mChannelMask = config.channel_mask;
                outputDesc->mFormat = config.format;

                for (size_t k = 0; k  < supportedDevices.size(); k++) {
                    ssize_t index = mAvailableOutputDevices.indexOf(supportedDevices[k]);
                    // give a valid ID to an attached device once confirmed it is reachable
                    if (index >= 0 && !mAvailableOutputDevices[index]->isAttached()) {
                        mAvailableOutputDevices[index]->attach(mHwModules[i]);
                    }
                }
                if (mPrimaryOutput == 0 &&
                        outProfile->getFlags() & AUDIO_OUTPUT_FLAG_PRIMARY) {
                    mPrimaryOutput = outputDesc;
                }
                addOutput(output, outputDesc);
                setOutputDevice(outputDesc,
                                outputDesc->mDevice,
                                true,
                                0,
                                NULL,
                                address.string());
            }
        }

		...

// config文件解析

#define AUDIO_POLICY_XML_CONFIG_FILE_PATH_MAX_LENGTH 128
#define AUDIO_POLICY_XML_CONFIG_FILE_NAME "audio_policy_configuration.xml"


#ifdef USE_XML_AUDIO_POLICY_CONF
// Treblized audio policy xml config will be located in /odm/etc or /vendor/etc.
static const char *kConfigLocationList[] =
        {"/odm/etc", "/vendor/etc/audio", "/vendor/etc", "/system/etc"};
static const int kConfigLocationListSize =
        (sizeof(kConfigLocationList) / sizeof(kConfigLocationList[0]));

static status_t deserializeAudioPolicyXmlConfig(AudioPolicyConfig &config) {
    char audioPolicyXmlConfigFile[AUDIO_POLICY_XML_CONFIG_FILE_PATH_MAX_LENGTH];
    status_t ret;

    for (int i = 0; i < kConfigLocationListSize; i++) {
        PolicySerializer serializer;
        snprintf(audioPolicyXmlConfigFile,
                 sizeof(audioPolicyXmlConfigFile),
                 "%s/%s",
                 kConfigLocationList[i],
                 AUDIO_POLICY_XML_CONFIG_FILE_NAME);
        ret = serializer.deserialize(audioPolicyXmlConfigFile, config);
        if (ret == NO_ERROR) {
            break;
        }
    }
    return ret;
}
#endif


audio_policy_configuration.xml文件
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!-- Copyright (c) 2016-2017, The Linux Foundation. All rights reserved
     Not a Contribution.
-->
<!-- Copyright (C) 2015 The Android Open Source Project

     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License.
-->

<audioPolicyConfiguration version="1.0" xmlns:xi="http://www.w3.org/2001/XInclude">
    <!-- version section contains a “version” tag in the form “major.minor” e.g version=”1.0” -->

    <!-- Global configuration Decalaration -->
    <globalConfiguration speaker_drc_enabled="true"/>


    <!-- Modules section:
        There is one section per audio HW module present on the platform.
        Each module section will contains two mandatory tags for audio HAL “halVersion” and “name”.
        The module names are the same as in current .conf file:
                “primary”, “A2DP”, “remote_submix”, “USB”
        Each module will contain the following sections:
		        “devicePorts”: a list of device descriptors for all input and output devices accessible via this
        module.
        This contains both permanently attached devices and removable devices.
        “mixPorts”: listing all output and input streams exposed by the audio HAL
        “routes”: list of possible connections between input and output devices or between stream and
        devices.
            "route": is defined by an attribute:
                -"type": <mux|mix> means all sources are mutual exclusive (mux) or can be mixed (mix)
                -"sink": the sink involved in this route
                -"sources": all the sources than can be connected to the sink via vis route
        “attachedDevices”: permanently attached devices.
        The attachedDevices section is a list of devices names. The names correspond to device names
        defined in <devicePorts> section.
        “defaultOutputDevice”: device to be used by default when no policy rule applies
    -->
    <modules>
        <!-- Primary Audio HAL -->
        <module name="primary" halVersion="2.0">
            <attachedDevices>
                <item>Earpiece</item>
                <item>Speaker</item>
                <item>Telephony Tx</item>
                <item>Built-In Mic</item>
                <item>Built-In Back Mic</item>
                <item>FM Tuner</item>
                <item>Telephony Rx</item>
            </attachedDevices>
            <defaultOutputDevice>Speaker</defaultOutputDevice>
            <mixPorts>
                <mixPort name="primary output" role="source" flags="AUDIO_OUTPUT_FLAG_FAST|AUDIO_OUTPUT_FLAG_PRIMARY
">
		                    <profile name="" format="AUDIO_FORMAT_PCM_16_BIT"
                             samplingRates="48000" channelMasks="AUDIO_CHANNEL_OUT_STEREO"/>
                </mixPort>
                <mixPort name="raw" role="source"
                        flags="AUDIO_OUTPUT_FLAG_FAST|AUDIO_OUTPUT_FLAG_RAW">
                    <profile name="" format="AUDIO_FORMAT_PCM_16_BIT"
                             samplingRates="48000" channelMasks="AUDIO_CHANNEL_OUT_STEREO"/>
                </mixPort>
                <mixPort name="deep_buffer" role="source"
                        flags="AUDIO_OUTPUT_FLAG_DEEP_BUFFER">
                    <profile name="" format="AUDIO_FORMAT_PCM_16_BIT"
                             samplingRates="48000" channelMasks="AUDIO_CHANNEL_OUT_STEREO"/>
                </mixPort>
                <mixPort name="direct_pcm" role="source"
                        flags="AUDIO_OUTPUT_FLAG_DIRECT">
                    <profile name="" format="AUDIO_FORMAT_PCM_16_BIT"
                             samplingRates="8000,11025,12000,16000,22050,24000,32000,44100,48000,64000,88200,96000,1
28000,176400,192000"
                             channelMasks="AUDIO_CHANNEL_OUT_MONO,AUDIO_CHANNEL_OUT_STEREO,AUDIO_CHANNEL_OUT_2POINT1
,AUDIO_CHANNEL_OUT_QUAD,AUDIO_CHANNEL_OUT_PENTA,AUDIO_CHANNEL_OUT_5POINT1,AUDIO_CHANNEL_OUT_6POINT1,AUDIO_CHANNEL_OU
T_7POINT1"/>
                    <profile name="" format="AUDIO_FORMAT_PCM_8_24_BIT"
                             samplingRates="8000,11025,12000,16000,22050,24000,32000,44100,48000,64000,88200,96000,1
28000,176400,192000"
                             channelMasks="AUDIO_CHANNEL_OUT_MONO,AUDIO_CHANNEL_OUT_STEREO,AUDIO_CHANNEL_OUT_2POINT1
,AUDIO_CHANNEL_OUT_QUAD,AUDIO_CHANNEL_OUT_PENTA,AUDIO_CHANNEL_OUT_5POINT1,AUDIO_CHANNEL_OUT_6POINT1,AUDIO_CHANNEL_OU
T_7POINT1"/>
                    <profile name="" format="AUDIO_FORMAT_PCM_24_BIT_PACKED"


    ...
<!-- include 包含其他的xml文件 -->
        <!-- A2dp Audio HAL -->
        <xi:include href="/vendor/etc/a2dp_audio_policy_configuration.xml"/>

        <!-- Usb Audio HAL -->
        <xi:include href="/vendor/etc/usb_audio_policy_configuration.xml"/>

        <!-- Remote Submix Audio HAL -->
        <xi:include href="/vendor/etc/r_submix_audio_policy_configuration.xml"/>

    </modules>
    <!-- End of Modules section -->

    <!-- Volume section -->

    <xi:include href="/vendor/etc/audio_policy_volumes.xml"/>
    <xi:include href="/vendor/etc/default_volume_tables.xml"/>
					
					
```
Shenzhen，2018-8-30