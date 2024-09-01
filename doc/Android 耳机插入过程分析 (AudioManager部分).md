```
接上一篇，记录audioManager对耳机插入的操作

https://www.cnblogs.com/helloworldtoyou/p/9868890.html

主要是发送广播，另外更新音频通路

初始化：
10-29 12:50:39.542  1400  1400 I SystemServer: StartAudioService
10-29 12:50:39.542  1400  1400 I SystemServiceManager: Starting com.android.server.audio.AudioService$Lifecycle
10-29 12:50:39.587  1400  1400 D AudioService: Master mono false
10-29 12:50:39.589  1400  1400 D AudioService: Master mute false, user=0
10-29 12:50:39.592  1400  1400 D AudioService: Mic mute false, user=0
10-29 12:50:39.684  1400  1400 D SystemServerTiming: StartAudioService took to complete: 142ms
10-29 12:50:40.614  1400  1706 D AudioService: Touch exploration enabled=false stream override delay is now 0 ms
10-29 12:50:40.614  1400  1706 D AudioService: Accessibility volume enabled = false
10-29 12:50:41.702  1400  1946 D AudioService: Volume policy changed: VolumePolicy[volumeDownToEnterSilent=true,volumeUpToExitSilent=true,doNotDisturbWhenSilent=true,vibrateToSilentDebounce=400]
10-29 12:50:41.802  1400  1621 D AudioService: Volume controller: VolumeController(android.os.BinderProxy@9c596d8,mVisible=false)
10-29 12:50:42.217  1400  1400 D AudioService: Master mono false
10-29 12:50:42.225  1400  1400 D AudioService: Master mute false, user=0
10-29 12:50:42.228  1400  1400 D AudioService: Mic mute false, user=0

插入：
11-07 11:39:55.551  2349  2349 I AudioService: setWiredDeviceConnectionState(1 nm:  addr:)
11-07 11:39:55.552  2349  2349 I AudioService: setWiredDeviceConnectionState(1 nm:  addr:)
11-07 11:39:55.552  2349  3588 I AudioService: onSetWiredDeviceConnectionState(dev:4 state:1 address: deviceName: caller: android);
11-07 11:39:55.552  2349  3588 I AudioService: handleDeviceConnection(true dev:4 address: name:)
11-07 11:39:55.552  2349  3588 I AudioService: deviceKey:0x4:
11-07 11:39:55.552  2349  3588 I AudioService: deviceSpec:null is(already)Connected:false
11-07 11:39:55.593  2349  3588 I AudioService: sendDeviceConnectionIntent(dev:0x4 state:0x1 address: name:);
11-07 11:39:55.594  2349  3588 I AudioService: updateAudioRoutes MAIN_HEADSET
11-07 11:39:55.594  2349  3588 I AudioService: updateAudioRoutes connType != 0, state: 1
11-07 11:39:55.594  2349  3588 I AudioService: updateAudioRoutes connType MSG_REPORT_NEW_ROUTES
11-07 11:39:55.594  2349  3588 I AudioService: onSetWiredDeviceConnectionState(dev:80000010 state:1 address: deviceName: caller: android);
11-07 11:39:55.594  2349  3588 I AudioService: handleDeviceConnection(true dev:80000010 address: name:)
11-07 11:39:55.594  2349  3588 I AudioService: deviceKey:0x80000010:
11-07 11:39:55.594  2349  3588 I AudioService: deviceSpec:null is(already)Connected:false
11-07 11:39:55.610  2349  3588 I AudioService: sendDeviceConnectionIntent(dev:0x80000010 state:0x1 address: name:);
11-07 11:39:55.610  2349  3588 I AudioService: updateAudioRoutes Liutao
11-07 11:39:55.611  2349  3588 I AudioService: onAccessoryPlugMediaUnmute newDevice=4 [headset]
11-07 11:39:55.611  2349  3588 I AudioService: mAudioHandler 5060, N: 3
11-07 11:39:55.618  2349  3588 I AudioService: onAccessoryPlugMediaUnmute newDevice=-2147483632 [-2147483632]

拔出；
11-07 11:41:08.165  2349  2349 I AudioService: setWiredDeviceConnectionState(0 nm:  addr:)
11-07 11:41:08.168  2349  2349 I AudioService: setWiredDeviceConnectionState(0 nm:  addr:)
11-07 11:41:08.870  2349  3588 I AudioService: onSetWiredDeviceConnectionState(dev:4 state:0 address: deviceName: caller: android);
11-07 11:41:08.876  2349  3588 I AudioService: handleDeviceConnection(false dev:4 address: name:)
11-07 11:41:08.876  2349  3588 I AudioService: deviceKey:0x4:
11-07 11:41:08.877  2349  3588 I AudioService: deviceSpec:[type:0x4 name: address:] is(already)Connected:true
11-07 11:41:08.885  2349  3588 I AudioService: sendDeviceConnectionIntent(dev:0x4 state:0x0 address: name:);
11-07 11:41:08.887  2349  3588 I AudioService: updateAudioRoutes MAIN_HEADSET
11-07 11:41:08.887  2349  3588 I AudioService: updateAudioRoutes connType != 0, state: 0
11-07 11:41:08.887  2349  3588 I AudioService: updateAudioRoutes connType MSG_REPORT_NEW_ROUTES
11-07 11:41:08.887  2349  3588 I AudioService: mAudioHandler 5060, N: 3
11-07 11:41:08.914  2349  3588 I AudioService: onSetWiredDeviceConnectionState(dev:80000010 state:0 address: deviceName: caller: android);
11-07 11:41:08.914  2349  3588 I AudioService: handleDeviceConnection(false dev:80000010 address: name:)
11-07 11:41:08.914  2349  3588 I AudioService: deviceKey:0x80000010:
11-07 11:41:08.914  2349  3588 I AudioService: deviceSpec:[type:0x80000010 name: address:] is(already)Connected:true
11-07 11:41:08.918  2349  3588 I AudioService: sendDeviceConnectionIntent(dev:0x80000010 state:0x0 address: name:);


frameworks\base\media\java\android\media\AudioManager.java
 public void setWiredDeviceConnectionState(int type, int state, String address, String name) {
        final IAudioService service = getService();
        try {
            service.setWiredDeviceConnectionState(type, state, address, name,
                    mApplicationContext.getOpPackageName());
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }
	
	
frameworks\base\services\core\java\com\android\server\audio\AudioService.java
	    public void setWiredDeviceConnectionState(int type, int state, String address, String name,
            String caller) {
        synchronized (mConnectedDevices) {
            if (DEBUG_DEVICES) {
                Slog.i(TAG, "setWiredDeviceConnectionState(" + state + " nm: " + name + " addr:"
                        + address + ")");
            }
            int delay = checkSendBecomingNoisyIntent(type, state, AudioSystem.DEVICE_NONE);
            queueMsgUnderWakeLock(mAudioHandler,
                    MSG_SET_WIRED_DEVICE_CONNECTION_STATE,
                    0 /* arg1 unused */,
                    0 /* arg2 unused */,
                    new WiredDeviceConnectionState(type, state, address, name, caller),
                    delay);
        }
    }
	
frameworks\base\services\core\java\com\android\server\audio\AudioService.java
	   private void queueMsgUnderWakeLock(Handler handler, int msg,
            int arg1, int arg2, Object obj, int delay) {
        final long ident = Binder.clearCallingIdentity();
        // Always acquire the wake lock as AudioService because it is released by the
        // message handler.
        mAudioEventWakeLock.acquire();
        Binder.restoreCallingIdentity(ident);
        sendMsg(handler, msg, SENDMSG_QUEUE, arg1, arg2, obj, delay);
    }
	
	
	    private static void sendMsg(Handler handler, int msg,
            int existingMsgPolicy, int arg1, int arg2, Object obj, int delay) {

        if (existingMsgPolicy == SENDMSG_REPLACE) {
            handler.removeMessages(msg);
        } else if (existingMsgPolicy == SENDMSG_NOOP && handler.hasMessages(msg)) {
            return;
        }
        synchronized (mLastDeviceConnectMsgTime) {
            long time = SystemClock.uptimeMillis() + delay;
            handler.sendMessageAtTime(handler.obtainMessage(msg, arg1, arg2, obj), time);
            if (msg == MSG_SET_WIRED_DEVICE_CONNECTION_STATE ||
                    msg == MSG_SET_A2DP_SRC_CONNECTION_STATE ||
                    msg == MSG_SET_A2DP_SINK_CONNECTION_STATE) {
                mLastDeviceConnectMsgTime = time;
            }
        }
    }
	
	
	   private class AudioHandler extends Handler {
	   
	                  case MSG_SET_WIRED_DEVICE_CONNECTION_STATE:
                    {   WiredDeviceConnectionState connectState =
                            (WiredDeviceConnectionState)msg.obj;
                        mWiredDevLogger.log(new WiredDevConnectEvent(connectState));
                        onSetWiredDeviceConnectionState(connectState.mType, connectState.mState,
                                connectState.mAddress, connectState.mName, connectState.mCaller);
                        mAudioEventWakeLock.release();
                    }
                    break;
	
	
	private void onSetWiredDeviceConnectionState(int device, int state, String address,
            String deviceName, String caller) {
        if (DEBUG_DEVICES) {
		
		//打印结果AudioService: onSetWiredDeviceConnectionState(dev:8 state:1 address: deviceName: caller: android);
            Slog.i(TAG, "onSetWiredDeviceConnectionState(dev:" + Integer.toHexString(device)
                    + " state:" + Integer.toHexString(state)
                    + " address:" + address
                    + " deviceName:" + deviceName
                    + " caller: " + caller + ");");
        }

        synchronized (mConnectedDevices) {
            if ((state == 0) && ((device & DEVICE_OVERRIDE_A2DP_ROUTE_ON_PLUG) != 0)) {
                setBluetoothA2dpOnInt(true, "onSetWiredDeviceConnectionState state 0");
            }
			// 打印AudioService: handleDeviceConnection(true dev:8 address: name:)
            if (!handleDeviceConnection(state == 1, device, address, deviceName)) {
                // change of connection state failed, bailout
                return;
            }
            if (state != 0) {
                if ((device & DEVICE_OVERRIDE_A2DP_ROUTE_ON_PLUG) != 0) {
                    setBluetoothA2dpOnInt(false, "onSetWiredDeviceConnectionState state not 0");
                }
                if ((device & mSafeMediaVolumeDevices) != 0) {
                    sendMsg(mAudioHandler,
                            MSG_CHECK_MUSIC_ACTIVE,
                            SENDMSG_REPLACE,
                            0,
                            0,
                            caller,
                            MUSIC_ACTIVE_POLL_PERIOD_MS);
                }
                // Television devices without CEC service apply software volume on HDMI output
                if (isPlatformTelevision() && ((device & AudioSystem.DEVICE_OUT_HDMI) != 0)) {
                    mFixedVolumeDevices |= AudioSystem.DEVICE_OUT_HDMI;
                    checkAllFixedVolumeDevices();
                    if (mHdmiManager != null) {
                        synchronized (mHdmiManager) {
                            if (mHdmiPlaybackClient != null) {
                                mHdmiCecSink = false;
                                mHdmiPlaybackClient.queryDisplayStatus(mHdmiDisplayStatusCallback);
                            }
                        }
                    }
                }
            } else {
                if (isPlatformTelevision() && ((device & AudioSystem.DEVICE_OUT_HDMI) != 0)) {
                    if (mHdmiManager != null) {
                        synchronized (mHdmiManager) {
                            mHdmiCecSink = false;
                        }
                    }
                }
            }
			// 发送耳机插入的广播
            sendDeviceConnectionIntent(device, state, address, deviceName);
            // 更新音频通路
			updateAudioRoutes(device, state);
        }
    }
	
	
	
	private void sendDeviceConnectionIntent(int device, int state, String address,
            String deviceName) {
        if (DEBUG_DEVICES) {
            Slog.i(TAG, "sendDeviceConnectionIntent(dev:0x" + Integer.toHexString(device) +
                    " state:0x" + Integer.toHexString(state) + " address:" + address +
                    " name:" + deviceName + ");");
        }
        Intent intent = new Intent();

        if (device == AudioSystem.DEVICE_OUT_WIRED_HEADSET) {
            intent.setAction(Intent.ACTION_HEADSET_PLUG);
            intent.putExtra("microphone", 1);
        } else if (device == AudioSystem.DEVICE_OUT_WIRED_HEADPHONE ||
                   device == AudioSystem.DEVICE_OUT_LINE) {
            intent.setAction(Intent.ACTION_HEADSET_PLUG);
            intent.putExtra("microphone",  0);
        } else if (device == AudioSystem.DEVICE_OUT_USB_HEADSET) {
            intent.setAction(Intent.ACTION_HEADSET_PLUG);
            intent.putExtra("microphone",
                    AudioSystem.getDeviceConnectionState(AudioSystem.DEVICE_IN_USB_HEADSET, "")
                        == AudioSystem.DEVICE_STATE_AVAILABLE ? 1 : 0);
        } else if (device == AudioSystem.DEVICE_IN_USB_HEADSET) {
            if (AudioSystem.getDeviceConnectionState(AudioSystem.DEVICE_OUT_USB_HEADSET, "")
                    == AudioSystem.DEVICE_STATE_AVAILABLE) {
                intent.setAction(Intent.ACTION_HEADSET_PLUG);
                intent.putExtra("microphone", 1);
            } else {
                // do not send ACTION_HEADSET_PLUG when only the input side is seen as changing
                return;
            }
        } else if (device == AudioSystem.DEVICE_OUT_HDMI ||
                device == AudioSystem.DEVICE_OUT_HDMI_ARC) {
            configureHdmiPlugIntent(intent, state);
        }

        if (intent.getAction() == null) {
            return;
        }

        intent.putExtra(CONNECT_INTENT_KEY_STATE, state);
        intent.putExtra(CONNECT_INTENT_KEY_ADDRESS, address);
        intent.putExtra(CONNECT_INTENT_KEY_PORT_NAME, deviceName);

        intent.addFlags(Intent.FLAG_RECEIVER_REGISTERED_ONLY);

        final long ident = Binder.clearCallingIdentity();
        // 发送广播，上层可以通过接受广播来判断耳机的状态。
        try {
            ActivityManager.broadcastStickyIntent(intent, UserHandle.USER_ALL);
        } finally {
            Binder.restoreCallingIdentity(ident);
        }
    }
	
	
	
	private void updateAudioRoutes(int device, int state)
    {
        int connType = 0;
        if (device == AudioSystem.DEVICE_OUT_WIRED_HEADSET) {
         //Slog.i(TAG, "updateAudioRoutes MAIN_HEADSET");进入这里
			connType = AudioRoutesInfo.MAIN_HEADSET;
        } else if (device == AudioSystem.DEVICE_OUT_WIRED_HEADPHONE ||
                   device == AudioSystem.DEVICE_OUT_LINE) {
            connType = AudioRoutesInfo.MAIN_HEADPHONES;
        } else if (device == AudioSystem.DEVICE_OUT_HDMI ||
                device == AudioSystem.DEVICE_OUT_HDMI_ARC) {
            connType = AudioRoutesInfo.MAIN_HDMI;
        } else if (device == AudioSystem.DEVICE_OUT_USB_DEVICE||
                device == AudioSystem.DEVICE_OUT_USB_HEADSET) {
            connType = AudioRoutesInfo.MAIN_USB;
        }

        synchronized (mCurAudioRoutes) {
            if (connType != 0) {
                int newConn = mCurAudioRoutes.mainType;
                if (state != 0) {
                    newConn |= connType;
                } else {
                    newConn &= ~connType;
                }
                if (newConn != mCurAudioRoutes.mainType) {
                    mCurAudioRoutes.mainType = newConn;
					// 发送消息给Handler
                    sendMsg(mAudioHandler, MSG_REPORT_NEW_ROUTES,
                            SENDMSG_NOOP, 0, 0, null, 0);
                }
            }
        }
    }
	
	
	private class AudioHandler extends Handler {
	   
	public void handleMessage(Message msg) {
	 case MSG_REPORT_NEW_ROUTES: {
                    int N = mRoutesObservers.beginBroadcast();
                    if (N > 0) {
                        AudioRoutesInfo routes;
                        synchronized (mCurAudioRoutes) {
                            routes = new AudioRoutesInfo(mCurAudioRoutes);
                        }
                        while (N > 0) {
                            N--;
                            IAudioRoutesObserver obs = mRoutesObservers.getBroadcastItem(N);
                            try {
                                obs.dispatchAudioRoutesChanged(routes);
                            } catch (RemoteException e) {
                            }
                        }
                    }
                    mRoutesObservers.finishBroadcast();
                    observeDevicesForStreams(-1);
                    break;
                }				

frameworks\base\media\java\android\media\MediaRouter.java
        final IAudioRoutesObserver.Stub mAudioRoutesObserver = new IAudioRoutesObserver.Stub() {
            @Override
            public void dispatchAudioRoutesChanged(final AudioRoutesInfo newRoutes) {
                mHandler.post(new Runnable() {
                    @Override public void run() {
                        updateAudioRoutes(newRoutes);
                    }
                });
            }
        };
```