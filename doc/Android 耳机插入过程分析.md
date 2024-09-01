Android 耳机插入过程分析

```
参考链接：
https://www.jianshu.com/p/d82a8dabb3e7

初始化：
10-26 07:40:43.932  1414  1414 I SystemServer: StartWiredAccessoryManager
10-26 07:40:43.936  1414  1414 W WiredAccessoryManager: This kernel does not have usb audio support
10-26 07:40:43.937  1414  1414 W WiredAccessoryManager: This kernel does not have HDMI audio support
10-26 07:40:43.937  1414  1414 W WiredAccessoryManager: This kernel does not have DP audio support
10-26 07:40:43.937  1414  1414 D SystemServerTiming: StartWiredAccessoryManager took to complete: 6ms
10-26 07:40:46.375  1414  1414 V WiredAccessoryManager: notifyWiredAccessoryChanged: when=0 bits= mask=54
10-26 07:40:46.375  1414  1414 V WiredAccessoryManager: newName=h2w newState=0 headsetState=0 prev headsetState=0
10-26 07:40:46.375  1414  1414 E WiredAccessoryManager: No state change.
10-26 07:40:46.375  1414  1414 V WiredAccessoryManager: init()

耳机插入
10-26 11:26:48.552  1414  1684 V WiredAccessoryManager: notifyWiredAccessoryChanged: when=13604972191000 bits=SW_HEADPHONE_INSERT SW_MICROPHONE_INSERT mask=94
10-26 11:26:48.552  1414  1684 V WiredAccessoryManager: newName=h2w newState=1 headsetState=1 prev headsetState=0
10-26 11:26:48.554  1414  1684 I WiredAccessoryManager: MSG_NEW_DEVICE_STATE
10-26 11:26:48.555  1414  1414 V WiredAccessoryManager: headsetName:  connected

耳机拔出
10-26 11:26:53.134  1414  1684 V WiredAccessoryManager: notifyWiredAccessoryChanged: when=13609554723000 bits= mask=94
10-26 11:26:53.134  1414  1684 V WiredAccessoryManager: newName=h2w newState=0 headsetState=0 prev headsetState=1
10-26 11:26:53.136  1414  1684 I WiredAccessoryManager: MSG_NEW_DEVICE_STATE
10-26 11:26:53.137  1414  1414 V WiredAccessoryManager: headsetName:  disconnected


高通读取外设插入方式有两种：
1. UEvent：
读取/sys/class/switch/h2w/state文件等状态
2. InputEvent：
读取/dev/input/event

通过属性值,判断采用何种方式。
frameworks\base\core\res\res\values\config.xml
<bool name="config_useDevInputEventForAudioJack">true</bool>

getevent
add device 1: /dev/input/event7
  name:     "msm8953-snd-card-mtp Button Jack"
add device 2: /dev/input/event6
  name:     "msm8953-snd-card-mtp Headset Jack"
add device 3: /dev/input/event2
  name:     "goodix-ts"
add device 4: /dev/input/event3
  name:     "hbtp_vm"
could not get driver version for /dev/input/mouse0, Not a typewriter
add device 5: /dev/input/event0
  name:     "qpnp_pon"
could not get driver version for /dev/input/mice, Not a typewriter
add device 6: /dev/input/event1
  name:     "tsu6721"
add device 7: /dev/input/event4
  name:     "keyremap_virtual"
add device 8: /dev/input/event5
  name:     "gpio-keys"
  
插入：
/dev/input/event6: 0005 0002 00000001
/dev/input/event6: 0005 0004 00000001
/dev/input/event6: 0005 0007 00000001
/dev/input/event6: 0000 0000 00000000

拔出：
/dev/input/event6: 0005 0002 00000000
/dev/input/event6: 0005 0004 00000000
/dev/input/event6: 0005 0007 00000000
/dev/input/event6: 0000 0000 00000000

getevent -l
/dev/input/event6: EV_SW        SW_HEADPHONE_INSERT  00000001
/dev/input/event6: EV_SW        SW_MICROPHONE_INSERT 00000001
/dev/input/event6: EV_SW        SW_JACK_PHYSICAL_INS 00000001
/dev/input/event6: EV_SYN       SYN_REPORT           00000000

/dev/input/event6: EV_SW        SW_HEADPHONE_INSERT  00000000
/dev/input/event6: EV_SW        SW_MICROPHONE_INSERT 00000000
/dev/input/event6: EV_SW        SW_JACK_PHYSICAL_INS 00000000
/dev/input/event6: EV_SYN       SYN_REPORT           00000000

代码跟踪
frameworks\base\services\core\java\com\android\server\input\InputManagerService.java
mUseDevInputEventForAudioJack =
                context.getResources().getBoolean(R.bool.config_useDevInputEventForAudioJack);
				

        if (mUseDevInputEventForAudioJack && (switchMask & SW_JACK_BITS) != 0) {
            mWiredAccessoryCallbacks.notifyWiredAccessoryChanged(whenNanos, switchValues,
                    switchMask);
        }
			
frameworks\base\services\core\java\com\android\server\WiredAccessoryManager.java
    public void notifyWiredAccessoryChanged(long whenNanos, int switchValues, int switchMask) {
        if (LOG) Slog.v(TAG, "notifyWiredAccessoryChanged: when=" + whenNanos
                + " bits=" + switchCodeToString(switchValues, switchMask)
                + " mask=" + Integer.toHexString(switchMask));

        synchronized (mLock) {
            int headset;
            mSwitchValues = (mSwitchValues & ~switchMask) | switchValues;
            switch (mSwitchValues &
                (SW_HEADPHONE_INSERT_BIT | SW_MICROPHONE_INSERT_BIT | SW_LINEOUT_INSERT_BIT)) {
                case 0:
                    headset = 0;
                    break;

                case SW_HEADPHONE_INSERT_BIT:
                    headset = BIT_HEADSET_NO_MIC;
                    break;

                case SW_LINEOUT_INSERT_BIT:
                    headset = BIT_LINEOUT;
                    break;

                case SW_HEADPHONE_INSERT_BIT | SW_MICROPHONE_INSERT_BIT:
                    headset = BIT_HEADSET;
                    break;

                case SW_MICROPHONE_INSERT_BIT:
                    headset = BIT_HEADSET;
                    break;

                default:
                    headset = 0;
                    break;
            }

            updateLocked(NAME_H2W,
                (mHeadsetState & ~(BIT_HEADSET | BIT_HEADSET_NO_MIC | BIT_LINEOUT)) | headset);
        }
    }
		
    private void updateLocked(String newName, int newState) {
        // Retain only relevant bits
        int headsetState = newState & SUPPORTED_HEADSETS;
        int usb_headset_anlg = headsetState & BIT_USB_HEADSET_ANLG;
        int usb_headset_dgtl = headsetState & BIT_USB_HEADSET_DGTL;
        int h2w_headset = headsetState & (BIT_HEADSET | BIT_HEADSET_NO_MIC | BIT_LINEOUT);
        boolean h2wStateChange = true;
        boolean usbStateChange = true;
        if (LOG) Slog.v(TAG, "newName=" + newName
                + " newState=" + newState
                + " headsetState=" + headsetState
                + " prev headsetState=" + mHeadsetState);

        if (mHeadsetState == headsetState) {
            Log.e(TAG, "No state change.");
            return;
        }

        // reject all suspect transitions: only accept state changes from:
        // - a: 0 headset to 1 headset
        // - b: 1 headset to 0 headset
        if (h2w_headset == (BIT_HEADSET | BIT_HEADSET_NO_MIC | BIT_LINEOUT)) {
            Log.e(TAG, "Invalid combination, unsetting h2w flag");
            h2wStateChange = false;
        }
        // - c: 0 usb headset to 1 usb headset
        // - d: 1 usb headset to 0 usb headset
        if (usb_headset_anlg == BIT_USB_HEADSET_ANLG && usb_headset_dgtl == BIT_USB_HEADSET_DGTL) {
            Log.e(TAG, "Invalid combination, unsetting usb flag");
            usbStateChange = false;
        }
        if (!h2wStateChange && !usbStateChange) {
            Log.e(TAG, "invalid transition, returning ...");
            return;
        }

        mWakeLock.acquire();
        // 发送数据给
        Log.i(TAG, "MSG_NEW_DEVICE_STATE");
        Message msg = mHandler.obtainMessage(MSG_NEW_DEVICE_STATE, headsetState,
                mHeadsetState, "");
        mHandler.sendMessage(msg);

        mHeadsetState = headsetState;
    }

    private final Handler mHandler = new Handler(Looper.myLooper(), null, true) {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_NEW_DEVICE_STATE:
                    setDevicesState(msg.arg1, msg.arg2, (String)msg.obj);
                    mWakeLock.release();
                    break;
                case MSG_SYSTEM_READY:
                    onSystemReady();
                    mWakeLock.release();
                    break;
            }
        }
    };

    private void setDevicesState(
            int headsetState, int prevHeadsetState, String headsetName) {
        synchronized (mLock) {
            int allHeadsets = SUPPORTED_HEADSETS;
            for (int curHeadset = 1; allHeadsets != 0; curHeadset <<= 1) {
                if ((curHeadset & allHeadsets) != 0) {
                    setDeviceStateLocked(curHeadset, headsetState, prevHeadsetState, headsetName);
                    allHeadsets &= ~curHeadset;
                }
            }
        }
    }

    private void setDeviceStateLocked(int headset,
            int headsetState, int prevHeadsetState, String headsetName) {
        if ((headsetState & headset) != (prevHeadsetState & headset)) {
            int outDevice = 0;
            int inDevice = 0;
            int state;

            if ((headsetState & headset) != 0) {
                state = 1;
            } else {
                state = 0;
            }

            if (headset == BIT_HEADSET) {
                outDevice = AudioManager.DEVICE_OUT_WIRED_HEADSET;
                inDevice = AudioManager.DEVICE_IN_WIRED_HEADSET;
            } else if (headset == BIT_HEADSET_NO_MIC){
                outDevice = AudioManager.DEVICE_OUT_WIRED_HEADPHONE;
            } else if (headset == BIT_LINEOUT){
                outDevice = AudioManager.DEVICE_OUT_LINE;
            } else if (headset == BIT_USB_HEADSET_ANLG) {
                outDevice = AudioManager.DEVICE_OUT_ANLG_DOCK_HEADSET;
            } else if (headset == BIT_USB_HEADSET_DGTL) {
                outDevice = AudioManager.DEVICE_OUT_DGTL_DOCK_HEADSET;
            } else if (headset == BIT_HDMI_AUDIO) {
                outDevice = AudioManager.DEVICE_OUT_HDMI;
            } else {
                Slog.e(TAG, "setDeviceState() invalid headset type: "+headset);
                return;
            }

            if (LOG) {
                Slog.v(TAG, "headsetName: " + headsetName +
                        (state == 1 ? " connected" : " disconnected"));
            }
    	    // 发送给audio manager.
            if (outDevice != 0) {
              mAudioManager.setWiredDeviceConnectionState(outDevice, state, "", headsetName);
            }
            if (inDevice != 0) {
              mAudioManager.setWiredDeviceConnectionState(inDevice, state, "", headsetName);
            }
        }
    }
```