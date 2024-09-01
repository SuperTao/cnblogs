记录一下蓝牙A2DP的流程
```


packages\apps\Settings\src\com\android\settings\bluetooth\BluetoothPairingDetail.java
    @Override
    void onDevicePreferenceClick(BluetoothDevicePreference btPreference) {
        disableScanning();
        super.onDevicePreferenceClick(btPreference);
    }   
    
packages\apps\Settings\src\com\android\settings\bluetooth\DeviceListPreferenceFragment.java
    void onDevicePreferenceClick(BluetoothDevicePreference btPreference) {
        btPreference.onClicked();
    }
        
packages\apps\Settings\src\com\android\settings\bluetooth\BluetoothDevicePreference.java
    void onClicked() {
        Context context = getContext();
        // 获取连接状态
        int bondState = mCachedDevice.getBondState();

        final MetricsFeatureProvider metricsFeatureProvider =
                FeatureFactory.getFactory(context).getMetricsFeatureProvider();
        // 已经连接
        if (mCachedDevice.isConnected()) {
            // 断开连接
            metricsFeatureProvider.action(context,
                    MetricsEvent.ACTION_SETTINGS_BLUETOOTH_DISCONNECT);
            askDisconnect();
            // 以前连接过，不需要再配对，直接进行连接
        } else if (bondState == BluetoothDevice.BOND_BONDED) {
            metricsFeatureProvider.action(context,
                    MetricsEvent.ACTION_SETTINGS_BLUETOOTH_CONNECT);
            mCachedDevice.connect(true);
            // 没连接过，进行配对，需要连接的双方都同意之后才能连接
        } else if (bondState == BluetoothDevice.BOND_NONE) {
            metricsFeatureProvider.action(context,
                    MetricsEvent.ACTION_SETTINGS_BLUETOOTH_PAIR);
            if (!mCachedDevice.hasHumanReadableName()) {
                metricsFeatureProvider.action(context,
                        MetricsEvent.ACTION_SETTINGS_BLUETOOTH_PAIR_DEVICES_WITHOUT_NAMES);
            }
            pair();
        }
    }
    
    

匹配过的设备，进行连接
    frameworks\base\packages\SettingsLib\src\com\android\settingslib\bluetooth\CachedBluetoothDevice.java
        public void connect(boolean connectAllProfiles) {
        // 是否配对过
        if (!ensurePaired()) {
            return;
        }

        mConnectAttempted = SystemClock.elapsedRealtime();
        connectWithoutResettingTimer(connectAllProfiles);
    }
    
        private void connectWithoutResettingTimer(boolean connectAllProfiles) {
        // Try to initialize the profiles if they were not.
        if (mProfiles.isEmpty()) {
            // if mProfiles is empty, then do not invoke updateProfiles. This causes a race
            // condition with carkits during pairing, wherein RemoteDevice.UUIDs have been updated
            // from bluetooth stack but ACTION.uuid is not sent yet.
            // Eventually ACTION.uuid will be received which shall trigger the connection of the
            // various profiles
            // If UUIDs are not available yet, connect will be happen
            // upon arrival of the ACTION_UUID intent.
            Log.d(TAG, "No profiles. Maybe we will connect later");
            return;
        }

        // Reset the only-show-one-error-dialog tracking variable
        mIsConnectingErrorPossible = true;

        int preferredProfiles = 0;
        for (LocalBluetoothProfile profile : mProfiles) {
            // connectAllProfile传进来的是true
            if (connectAllProfiles ? profile.isConnectable() : profile.isAutoConnectable()) {
                if (profile.isPreferred(mDevice)) {
                    ++preferredProfiles;
                    connectInt(profile);			// 选择对应的profile进行连接
                }
            }
        }
        if (DEBUG) Log.d(TAG, "Preferred profiles = " + preferredProfiles);

        if (preferredProfiles == 0) {
            connectAutoConnectableProfiles();
        }
    }

    private void connectAutoConnectableProfiles() {
        if (!ensurePaired()) {
            return;
        }
        // Reset the only-show-one-error-dialog tracking variable
        mIsConnectingErrorPossible = true;

        for (LocalBluetoothProfile profile : mProfiles) {
            if (profile.isAutoConnectable()) {
                profile.setPreferred(mDevice, true);
                connectInt(profile);
            }
        }
    }
	
frameworks\base\packages\SettingsLib\src\com\android\settingslib\bluetooth\CachedBluetoothDevice.java
	    synchronized void connectInt(LocalBluetoothProfile profile) {
		// 判断是否配对过
        if (!ensurePaired()) {
            return;
        }
		// 连接
        if (profile.connect(mDevice)) {
            if (Utils.D) {
                Log.d(TAG, "Command sent successfully:CONNECT " + describe(profile));
            }
            return;
        }
        Log.i(TAG, "Failed to connect " + profile.toString() + " to " + mName);
    }
	

A2dp的profile	
frameworks\base\packages\SettingsLib\src\com\android\settingslib\bluetooth\A2dpProfile.java
public class A2dpProfile implements LocalBluetoothProfile {

    public boolean connect(BluetoothDevice device) {
        if (mService == null) return false;
        List<BluetoothDevice> sinks = getConnectedDevices();
        if (sinks != null) {
            for (BluetoothDevice sink : sinks) {
                if (sink.equals(device)) {
                    Log.w(TAG, "Connecting to device " + device + " : disconnect skipped");
                    continue;
                }
            }
        }
		// 连接
        return mService.connect(device);
    }
	
	
frameworks\base\core\java\android\bluetooth\BluetoothA2dp.java
	 @GuardedBy("mServiceLock") private IBluetoothA2dp mService;
	 
	public boolean connect(BluetoothDevice device) {
        if (DBG) log("connect(" + device + ")");
        try {
            mServiceLock.readLock().lock();
            if (mService != null && isEnabled() &&
                isValidDevice(device)) {
                return mService.connect(device);	// 调用aidl中的方法
            }
            if (mService == null) Log.w(TAG, "Proxy not attached to service");
            return false;
        } catch (RemoteException e) {
            Log.e(TAG, "Stack:" + Log.getStackTraceString(new Throwable()));
            return false;
        } finally {
            mServiceLock.readLock().unlock();
        }
    }
	
	
	
Binder进程间通信，服务端实现方法如下。	
packages\apps\Bluetooth\src\com\android\bluetooth\a2dp\A2dpService.java
	private static class BluetoothA2dpBinder extends IBluetoothA2dp.Stub 
    implements IProfileServiceBinder {
		
	        public boolean connect(BluetoothDevice device) {
            A2dpService service = getService();
            if (service == null) return false;
            //do not allow new connections with active multicast
            if (service.isMulticastOngoing(device)) {
                Log.i(TAG,"A2dp Multicast is Ongoing, ignore Connection Request");
                return false;
            }
            return service.connect(device);
        }


    public boolean connect(BluetoothDevice device) {
        if (DBG) Log.d(TAG, "Enter connect");
        enforceCallingOrSelfPermission(BLUETOOTH_ADMIN_PERM,
                                       "Need BLUETOOTH ADMIN permission");

        if (getPriority(device) == BluetoothProfile.PRIORITY_OFF) {
            return false;
        }
        ParcelUuid[] featureUuids = device.getUuids();
        if ((BluetoothUuid.containsAnyUuid(featureUuids, A2DP_SOURCE_UUID)) &&
            !(BluetoothUuid.containsAllUuids(featureUuids ,A2DP_SOURCE_SINK_UUIDS))) {
            Log.e(TAG,"Remote does not have A2dp Sink UUID");
            return false;
        }

        int connectionState = BluetoothProfile.STATE_DISCONNECTED;
        synchronized(mBtA2dpLock) {
            if (mStateMachine != null) {
                connectionState = mStateMachine.getConnectionState(device);
            }
        }
        if (connectionState == BluetoothProfile.STATE_CONNECTED ||
            connectionState == BluetoothProfile.STATE_CONNECTING) {
            return false;
        }
		// 发送消息给状态机
        synchronized(mBtA2dpLock) {
            if (mStateMachine != null) {
                mStateMachine.sendMessage(A2dpStateMachine.CONNECT, device);
            }
        }
        if (DBG) Log.d(TAG, "Exit connect");
        return true;
    }		
	
	
	状态机初始状态Disconnect
packages\apps\Bluetooth\src\com\android\bluetooth\a2dp\A2dpStateMachine.java
final class A2dpStateMachine extends StateMachine {
	
	
	           switch(message.what) {
                case CONNECT:
                    BluetoothDevice device = (BluetoothDevice) message.obj;
                    // 发送广播，状态改变
                    broadcastConnectionState(device, BluetoothProfile.STATE_CONNECTING,
                                   BluetoothProfile.STATE_DISCONNECTED);
                    // 进行连接
                    if (!connectA2dpNative(getByteAddress(device)) ) {
						// 连接完成，失败就再次发广播，状态有connecting->disconnect 
                        broadcastConnectionState(device, BluetoothProfile.STATE_DISCONNECTED,
                                       BluetoothProfile.STATE_CONNECTING);
                        break;
                    }

                    synchronized (A2dpStateMachine.this) {
                        mTargetDevice = device;
                        transitionTo(mPending);         // 切换状态机状态
                    }
                    // TODO(BT) remove CONNECT_TIMEOUT when the stack
                    //          sends back events consistently
                    Message m = obtainMessage(CONNECT_TIMEOUT);
                    m.obj = device;
                    sendMessageDelayed(m, CONNECT_TIMEOUT_SEC);
                    break;
					
					
packages\apps\Bluetooth\jni\com_android_bluetooth_a2dp.cpp
static jboolean connectA2dpNative(JNIEnv* env, jobject object,
                                  jbyteArray address) {
  ALOGI("%s: sBluetoothA2dpInterface: %p", __func__, sBluetoothA2dpInterface);
  if (!sBluetoothA2dpInterface) return JNI_FALSE;

  jbyte* addr = env->GetByteArrayElements(address, NULL);
  if (!addr) {
    jniThrowIOException(env, EINVAL);
    return JNI_FALSE;
  }

  bt_status_t status = sBluetoothA2dpInterface->connect((RawAddress*)addr);
  if (status != BT_STATUS_SUCCESS) {
    ALOGE("Failed HF connection, status: %d", status);
  }
  env->ReleaseByteArrayElements(address, addr, 0);
  return (status == BT_STATUS_SUCCESS) ? JNI_TRUE : JNI_FALSE;
}

初始化
static void initNative(JNIEnv* env, jobject object,
                       jobjectArray codecConfigArray,
                       jint maxA2dpConnection,
                       jint multiCastState) {
					   
 sBluetoothA2dpInterface =
      (btav_source_interface_t*)btInf->get_profile_interface(
          BT_PROFILE_ADVANCED_AUDIO_ID);
  if (sBluetoothA2dpInterface == NULL) {
    ALOGE("Failed to get Bluetooth A2DP Interface");
    return;
  }
  
  
  
system\bt\btif\src\bluetooth.cc
  static const void* get_profile_interface(const char* profile_id) {
  LOG_INFO(LOG_TAG, "%s: id = %s", __func__, profile_id);

  /* sanity check */
  if (interface_ready() == false) return NULL;

  /* check for supported profile interfaces */
  if (is_profile(profile_id, BT_PROFILE_HANDSFREE_ID))
    return btif_hf_get_interface();

  if (is_profile(profile_id, BT_PROFILE_HANDSFREE_CLIENT_ID))
    return btif_hf_client_get_interface();

  if (is_profile(profile_id, BT_PROFILE_SOCKETS_ID))
    return btif_sock_get_interface();

  if (is_profile(profile_id, BT_PROFILE_PAN_ID))
    return btif_pan_get_interface();
	// audio有两个，一个是作为source用
  if (is_profile(profile_id, BT_PROFILE_ADVANCED_AUDIO_ID))
    return btif_av_get_src_interface();
	// audio， 作为sink用
  if (is_profile(profile_id, BT_PROFILE_ADVANCED_AUDIO_SINK_ID))
    return btif_av_get_sink_interface();
	
	
	
system\bt\btif\src\btif_av.cc
const btav_source_interface_t* btif_av_get_src_interface(void) {
  BTIF_TRACE_EVENT("%s", __func__);
  return &bt_av_src_interface;
}

static const btav_source_interface_t bt_av_src_interface = {
    sizeof(btav_source_interface_t),
    init_src,
    src_connect_sink,	// 音频源连接sink，这个应该是对音频进行编码，然后输出
    disconnect,
    codec_config_src,
    cleanup_src,
    allow_connection,
    select_audio_device,
};

static const btav_sink_interface_t bt_av_sink_interface = {
    sizeof(btav_sink_interface_t),
    init_sink,
    sink_connect_src,	// sink连接音频源， sink接受音频数据，解码进行播放
    disconnect,
    cleanup_sink,
    update_audio_focus_state,
    update_audio_track_gain,
};



连接成功之后，底层发送事件。
    packages\apps\Bluetooth\src\com\android\bluetooth\a2dp\A2dpStateMachine.java
	private void onConnectionStateChanged(int state, byte[] address) {
        log("Enter onConnectionStateChanged() ");
        StackEvent event = new StackEvent(EVENT_TYPE_CONNECTION_STATE_CHANGED);
        event.valueInt = state;
        event.device = getDevice(address);
        sendMessage(STACK_EVENT, event);
        log("Exit onConnectionStateChanged() ");
    }
	
	
	private class Disconnected extends State {
	
                case STACK_EVENT:
                    StackEvent event = (StackEvent) message.obj;
                    switch (event.type) {
                        case EVENT_TYPE_CONNECTION_STATE_CHANGED:
                            processConnectionEvent(event.valueInt, event.device);
                            break;
                        default:
                            loge("Unexpected stack event: " + event.type);
                            break;
	
       private void processConnectionEvent(int state, BluetoothDevice device) {
            log("processConnectionEvent state = " + state +
                    ", device = " + device);
            switch (state) {
            ...
            case CONNECTION_STATE_CONNECTED:
                logw("A2DP Connected from Disconnected state");
                if (okToConnect(device)) {
                    logi("Incoming A2DP accepted");
                    synchronized (A2dpStateMachine.this) {
                        if (!mConnectedDevicesList.contains(device)) {
                            mConnectedDevicesList.add(device);
                            log( "device " + device.getAddress() +
                                    " is adding in Disconnected state");
                        }
                        mCurrentDevice = device;
                        transitionTo(mConnected);
                    }
                    broadcastConnectionState(device, BluetoothProfile.STATE_CONNECTED,
                                             BluetoothProfile.STATE_DISCONNECTED);
                } else {
                    //reject the connection and stay in Disconnected state itself
                    logi("Incoming A2DP rejected");
                    disconnectA2dpNative(getByteAddress(device));
                }
                break;	

				    private void broadcastConnectionState(BluetoothDevice device, int newState, int prevState) {
        log("Enter broadcastConnectionState() ");
        int delay = 0;
        if (mDummyDevice == null) {
           Log.i(TAG, "Setting the dummy device for audio service: " + device);
           String dummyAddress = "FA:CE:FA:CE:FA:CE";
           mDummyDevice = mAdapter.getRemoteDevice(dummyAddress);
        }

        if ((newState == BluetoothProfile.STATE_CONNECTED) ||
            (newState == BluetoothProfile.STATE_DISCONNECTING)) {
            if (mConnectedDevicesList.size() == 1) {
                Log.d("A2dpStateMachine", "broadcasting connection state");
                delay = mAudioManager.setBluetoothA2dpDeviceConnectionState(mDummyDevice,
                                                       newState, BluetoothProfile.A2DP);
            } else {
                Log.d("A2dpStateMachine", "DualA2dp: not broadcasting connected/disconnecting state");
            }
        } else if ((newState == BluetoothProfile.STATE_DISCONNECTED) ||
                   (newState == BluetoothProfile.STATE_CONNECTING)) {
            if (mConnectedDevicesList.size() == 0) {
                Log.d("A2dpStateMachine", "broadcasting connection state");
                delay = mAudioManager.setBluetoothA2dpDeviceConnectionState(mDummyDevice,
                                                       newState, BluetoothProfile.A2DP);
            } else {
                Log.d("A2dpStateMachine", "DualA2dp: not broadcasting connecting/disconnected state");
            }
        }

        if (mCodecNotifPending && newState == BluetoothProfile.STATE_CONNECTED) {
            Log.i(TAG, "There was pending codec config change, dispatching now. " + "device: " + device);
            Log.i(TAG, "Sending broadcast with device " + mDummyDevice);

            Intent intent = new Intent(BluetoothA2dp.ACTION_CODEC_CONFIG_CHANGED);
            intent.putExtra(BluetoothCodecStatus.EXTRA_CODEC_STATUS, mCodecStatus);
            intent.addFlags(Intent.FLAG_RECEIVER_REGISTERED_ONLY_BEFORE_BOOT);
            intent.addFlags(Intent.FLAG_RECEIVER_INCLUDE_BACKGROUND);
            intent.addFlags(Intent.FLAG_RECEIVER_FOREGROUND);

            intent.putExtra(BluetoothDevice.EXTRA_DEVICE, mDummyDevice);
            mAudioManager.handleBluetoothA2dpDeviceConfigChange(mDummyDevice);
            mCodecNotifPending = false;

            mContext.sendBroadcast(intent, A2dpService.BLUETOOTH_PERM);
        }

        Log.i(TAG,"mLastDelay: " + mLastDelay + " Current_Delay: " + delay);
        if (mIntentBroadcastHandler.hasMessages(MSG_CONNECTION_STATE_CHANGED)) {
            Log.i(TAG," Braodcast handler has the pending messages: " );
            if (mLastDelay > delay) {
                Log.i(TAG,"Last delay is greater than the current delay: " );
                delay = mLastDelay;
            }
        }
        mLastDelay = delay;

        Log.i(TAG,"connection state change: " + device + " newState: " + newState + " prevState:" + prevState);

        mWakeLock.acquire();
		// 发送广播
        mIntentBroadcastHandler.sendMessageDelayed(mIntentBroadcastHandler.obtainMessage(
            MSG_CONNECTION_STATE_CHANGED, prevState, newState, device), delay);
        log("Exit broadcastConnectionState() ");
    }
	broadcastConnectionState中会向AudioManager中设置A2DP的连接状态，返回值用来延时发送广播。AudioManager设置A2DP的连接状态非常重要，这样音频系统根据当前状态，判断音频从哪里发出（蓝牙a2dp、扬声器、耳机）。
	
```
Liu Tao
2019-3-28