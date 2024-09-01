记录一下wifi初始化过程。
```
packages/apps/Settings/src/com/android/settings/wifi/WifiSettings.java
public void onStart() {
        super.onStart();
//创建WifiEnabler对象
        // On/off switch is hidden for Setup Wizard (returns null)
        mWifiEnabler = createWifiEnabler();


packages/apps/Settings/src/com/android/settings/wifi/WifiEnabler.java
// wifi开关
public boolean onSwitchToggled(boolean isChecked)
{
	mWifiManager.setWifiEnabled(isChecked


根据wifi开关打开wifi.
frameworks/base/wifi/java/android/wifi/Wifimanager.java
    public boolean setWifiEnabled(boolean enabled) {
        try {
            return mService.setWifiEnabled(mContext.getOpPackageName(), enabled);
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }


WifiManger使用aidl和WifiService进行通行
frameworks/base/wifi/java/android/wifi/IWifiManager.aidl
boolean setWifiEnabled(String packageName, boolean enable);

// WifiService与WifiManager通信
frameworks/opt/net/wifi/service/java/com/android/server/wifi/WifiService.java
    public WifiService(Context context) {
        super(context);
        mImpl = new WifiServiceImpl(context, new WifiInjector(context), new WifiAsyncChannel(TAG));
    }
frameworks\opt\net\wifi\service\java\com\android\server\wifi\WifiServiceImpl.java	
WifiServiceImpl中实现WifiService的方法

    public synchronized boolean setWifiEnabled(String packageName, boolean enable)
            throws RemoteException {
        mWifiController.sendMessage(CMD_WIFI_TOGGLED);
        return true;
		
调用WifiControll状态机发送消息

WifiControll接受消息并处理。	
frameworks/opt/net/wifi/service/java/com/android/server/wifi/WifiController.java
	WifiController状态机处理消息：CMD_WIFI_TOGGLED
WifiController(Context context, WifiStateMachine wsm, WifiSettingsStore wss,
            WifiLockManager wifiLockManager, Looper looper, FrameworkFacade f,
            WifiApConfigStore wacs) {
        super(TAG, looper);
        mFacade = f;
        mContext = context;
        mWifiStateMachine = wsm;
        mSettingsStore = wss;
        mWifiLockManager = wifiLockManager;
        mWifiApConfigStore = wacs;

        mAlarmManager = (AlarmManager)mContext.getSystemService(Context.ALARM_SERVICE);
        Intent idleIntent = new Intent(ACTION_DEVICE_IDLE, null);
        mIdleIntent = mFacade.getBroadcast(mContext, IDLE_REQUEST, idleIntent, 0);
		// 初始化状态机，并添加状态
        addState(mDefaultState);
            addState(mApStaDisabledState, mDefaultState);
            addState(mStaEnabledState, mDefaultState);
            addState(mQcStaEnablingState, mDefaultState);
            addState(mQcStaDisablingState, mDefaultState);
            addState(mQcApEnablingState, mDefaultState);
            addState(mQcApDisablingState, mDefaultState);
            addState(mQcApStaEnablingState, mDefaultState);
            addState(mQcApStaDisablingState, mDefaultState);
            addState(mQcApStaEnabledState, mDefaultState);
                addState(mDeviceActiveState, mStaEnabledState);
                addState(mDeviceInactiveState, mStaEnabledState);
                    addState(mScanOnlyLockHeldState, mDeviceInactiveState);
                    addState(mFullLockHeldState, mDeviceInactiveState);
                    addState(mFullHighPerfLockHeldState, mDeviceInactiveState);
                    addState(mNoLockHeldState, mDeviceInactiveState);
            addState(mStaDisabledWithScanState, mDefaultState);
            addState(mApEnabledState, mDefaultState);
            addState(mEcmState, mDefaultState);
		// 判断飞行模式，wifi是否使能，是否打开了location wifi
        boolean isAirplaneModeOn = mSettingsStore.isAirplaneModeOn();
        boolean isWifiEnabled = mSettingsStore.isWifiToggleEnabled();
        boolean isScanningAlwaysAvailable = mSettingsStore.isScanAlwaysAvailable();

        log("isAirplaneModeOn = " + isAirplaneModeOn +
                ", isWifiEnabled = " + isWifiEnabled +
                ", isScanningAvailable = " + isScanningAlwaysAvailable);
        // 设置wifi状态机的初始状态, isScanningAlwaysAvailable是打开了location wifi服务
        if (isScanningAlwaysAvailable) {
            setInitialState(mStaDisabledWithScanState);
        } else {
            setInitialState(mApStaDisabledState);
        }
		       setLogRecSize(100);
        setLogOnlyTransitions(false);
		// 添加广播接收者，用于接收广播
        IntentFilter filter = new IntentFilter();
        filter.addAction(ACTION_DEVICE_IDLE);
        filter.addAction(WifiManager.NETWORK_STATE_CHANGED_ACTION);
        filter.addAction(WifiManager.WIFI_AP_STATE_CHANGED_ACTION);
        filter.addAction(WifiManager.WIFI_STATE_CHANGED_ACTION);
        mContext.registerReceiver(
                new BroadcastReceiver() {
                    @Override
                    public void onReceive(Context context, Intent intent) {
                        String action = intent.getAction();
                        if (action.equals(ACTION_DEVICE_IDLE)) {
                            sendMessage(CMD_DEVICE_IDLE);
                        } else if (action.equals(WifiManager.NETWORK_STATE_CHANGED_ACTION)) {
                            mNetworkInfo = (NetworkInfo) intent.getParcelableExtra(
                                    WifiManager.EXTRA_NETWORK_INFO);
                        } else if (action.equals(WifiManager.WIFI_AP_STATE_CHANGED_ACTION)) {
                            int state = intent.getIntExtra(
                                    WifiManager.EXTRA_WIFI_AP_STATE,
                                    WifiManager.WIFI_AP_STATE_FAILED);
                            if (state == WifiManager.WIFI_AP_STATE_FAILED) {
                                loge(TAG + "SoftAP start failed");
                                sendMessage(CMD_AP_START_FAILURE);
                            } else if (state == WifiManager.WIFI_AP_STATE_DISABLED) {
                                sendMessage(CMD_AP_STOPPED);
                            } else if (state == WifiManager.WIFI_AP_STATE_ENABLED) {
                                sendMessage(CMD_AP_STARTED);
                            }
                        } else if (action.equals(WifiManager.WIFI_STATE_CHANGED_ACTION)) {
                            int state = intent.getIntExtra(
                                    WifiManager.EXTRA_WIFI_STATE,
                                    WifiManager.WIFI_STATE_UNKNOWN);
                            if (state == WifiManager.WIFI_STATE_UNKNOWN) {
                                loge(TAG + "Wifi turn on failed");
                                sendMessage(CMD_STA_START_FAILURE);
                            } else if (state == WifiManager.WIFI_STATE_ENABLED) {
                                sendMessage(CMD_WIFI_ENABLED);
                            } else if (state == WifiManager.WIFI_STATE_DISABLED) {
                                sendMessage(CMD_WIFI_DISABLED);
                            }
                        }
                    }
                },
                new IntentFilter(filter));

        initializeAndRegisterForSettingsChange(looper);
    }
// 先查看没有打开location wifi的情况
// 初始状态，切换状态首先进入enter函数	
    class ApStaDisabledState extends State {
        private int mDeferredEnableSerialNumber = 0;
        private boolean mHaveDeferredEnable = false;
        private long mDisabledTimestamp;

        @Override
        public void enter() {
            /*
             * For STA+SAP concurrency request to stop supplicant is
             * handled in QcStaDisablingState. Hence skip for standalone case.
             */
			 // 判断STA和AP是否并发
            if (!mStaAndApConcurrency) {
                mWifiStateMachine.setSupplicantRunning(false);   // 关闭 wpa supplicant, 关闭的会延时一下在发送关闭的消息。
                // Supplicant can't restart right away, so not the time we switched off
                mDisabledTimestamp = SystemClock.elapsedRealtime();
                mDeferredEnableSerialNumber++;
                mHaveDeferredEnable = false;		// 将消息设置为不可推迟。
               mWifiStateMachine.clearANQPCache();
            }
        }
        @Override
        public boolean processMessage(Message msg) {
            logStateAndMessage(msg, this);
            switch (msg.what) {
                case CMD_WIFI_TOGGLED:		// 打开wifi发送的是CMD_WIFI_TOGGLED，这里处理这个消息
                case CMD_AIRPLANE_TOGGLED:
                    if (mSettingsStore.isWifiToggleEnabled()) {
                        if (doDeferEnable(msg)) {    // 判断是否使能了消息保留，没到就进入if
                            if (mHaveDeferredEnable) {
                                //  have 2 toggles now, inc serial number an ignore both
                                mDeferredEnableSerialNumber++;
                            }
                            mHaveDeferredEnable = !mHaveDeferredEnable;
                            break;
                        }
                        if (mDeviceIdle == false) {
                            if (mStaAndApConcurrency) {     // STA和AP并发
                                transitionTo(mQcStaEnablingState);  // 切换到QcStaEnablingState状态
                            } else {
                                // wifi is toggled, we need to explicitly tell WifiStateMachine that we
                                // are headed to connect mode before going to the DeviceActiveState
                                // since that will start supplicant and WifiStateMachine may not know
                                // what state to head to (it might go to scan mode).
                                mWifiStateMachine.setOperationalMode(WifiStateMachine.CONNECT_MODE);
                                // 切换到DeviceActiveState.
                                // 根据状态机的层级关系，需要先切换到其父类StaEnabledState,然后再切换到DeviceActiveState.
                                transitionTo(mDeviceActiveState);
                            }
                        } else {
                            checkLocksAndTransitionWhenDeviceIdle();
                        }
                    } else if (mSettingsStore.isScanAlwaysAvailable()) {
                        transitionTo(mStaDisabledWithScanState);
                    }
                    break;
                case CMD_SCAN_ALWAYS_MODE_CHANGED:
                    if (mSettingsStore.isScanAlwaysAvailable()) {       // loction wifi 打开，切换状态
                        transitionTo(mStaDisabledWithScanState);
                    }
                    break;
                case CMD_SET_AP:
                    if (msg.arg1 == 1) {
                        if (msg.arg2 == 0) { // previous wifi state has not been saved yet
                            mSettingsStore.setWifiSavedState(WifiSettingsStore.WIFI_DISABLED);
                        }

                        /* For STA+SAP concurrency request to start SoftAP is handled in QcApEnablingState. */
                        if (mStaAndApConcurrency) {
                            transitionTo(mQcApEnablingState);
                        } else {
                            mWifiStateMachine.setHostApRunning((SoftApModeConfiguration) msg.obj,
                                    true);
                            transitionTo(mApEnabledState);
                        }
                    }
                    break;
                case CMD_DEFERRED_TOGGLE:
                    if (msg.arg1 != mDeferredEnableSerialNumber) {
                        log("DEFERRED_TOGGLE ignored due to serial mismatch");
                        break;
                    }
                    log("DEFERRED_TOGGLE handled");
                    sendMessage((Message)(msg.obj));
                    break;
                case CMD_RESTART_WIFI_CONTINUE:
                    if (mStaAndApConcurrency) {
                        log("ApStaDisabledState: CMD_RESTART_WIFI_CONTINUE -> mQcStaEnablingState");
                        if (mRestartStaSapStack) {
                            deferMessage(msg);
                        }
                        transitionTo(mQcStaEnablingState);
                    } else {
                        transitionTo(mDeviceActiveState);
                    }
                    break;
                default:
                    return NOT_HANDLED;
            }
            return HANDLED;
        
	}

当WIFI的STA和AP都是关闭，并且没有并发，打开wifi开关，有上面可知，进入StaEnabledState状态。
class StaEnabledState extends State {
        @Override
        public void enter() {
            /*
             * For STA+SAP concurrency request to start supplicant is
             * handled in QcStaEnablingState. Hence skip for standalone case
             */
            // WifiStateMachine启动wpa_supplicant
            if (!mStaAndApConcurrency) {
                mWifiStateMachine.setSupplicantRunning(true);
            }
        }
        @Override
        public boolean processMessage(Message msg) {
            logStateAndMessage(msg, this);
            switch (msg.what) {
                case CMD_WIFI_TOGGLED:
                    if (! mSettingsStore.isWifiToggleEnabled()) {
                        if (mSettingsStore.isScanAlwaysAvailable()) {
                            transitionTo(mStaDisabledWithScanState);
                        } else {
                            if (mStaAndApConcurrency) {
                                transitionTo(mQcStaDisablingState);
                            } else {
                                transitionTo(mApStaDisabledState);
                            }
                        }
                    }
                    break;
                case CMD_AIRPLANE_TOGGLED:
                    /* When wi-fi is turned off due to airplane,
                    * disable entirely (including scan)
                    */
                    if (!mSettingsStore.isWifiToggleEnabled()) {
                        if (mStaAndApConcurrency) {
                            transitionTo(mQcStaDisablingState);
                        } else {
                            transitionTo(mApStaDisabledState);
                        }
                    }
                    break;
                case CMD_STA_START_FAILURE:
                    if (!mSettingsStore.isScanAlwaysAvailable()) {
                        transitionTo(mApStaDisabledState);
                    } else {
                        transitionTo(mStaDisabledWithScanState);
                    }
                    break;
                case CMD_EMERGENCY_CALL_STATE_CHANGED:
                case CMD_EMERGENCY_MODE_CHANGED:
                    boolean getConfigWiFiDisableInECBM = mFacade.getConfigWiFiDisableInECBM(mContext);
                    log("WifiController msg " + msg + " getConfigWiFiDisableInECBM "
                            + getConfigWiFiDisableInECBM);
                    if ((msg.arg1 == 1) && getConfigWiFiDisableInECBM) {
                        transitionTo(mEcmState);
                    }
                    break;
                case CMD_SET_AP:
                    if (msg.arg1 == 1) {
                        if (mStaAndApConcurrency) {
                            Slog.d(TAG,"StaEnabledState:CMD_SET_AP:setHostApRunning(true)-> mApStaEnableState");
                            mSoftApStateMachine.setHostApRunning(((SoftApModeConfiguration) msg.obj), true);
                            transitionTo(mQcApStaEnablingState);
                        } else {
                            // remeber that we were enabled
                            mSettingsStore.setWifiSavedState(WifiSettingsStore.WIFI_ENABLED);
                            deferMessage(obtainMessage(msg.what, msg.arg1, 1, msg.obj));
                            transitionTo(mApStaDisabledState);
                        }
                    }
                    break;
                case CMD_RESTART_WIFI:
                    if (mStaAndApConcurrency) {
                        log("StaEnabledState:CMD_RESTART_WIFI ->QcStaDisablingState");
                        deferMessage(obtainMessage(CMD_RESTART_WIFI_CONTINUE));
                        transitionTo(mQcStaDisablingState);
                    }
                    break;
                case CMD_RESTART_WIFI_CONTINUE:
                    if (mStaAndApConcurrency && mRestartStaSapStack) {
                        if (msg.obj != null) {
                            log("StaEnabledState:CMD_RESTART_WIFI_CONTINUE ->mQcApStaEnablingState");
                            mSoftApStateMachine.setHostApRunning(((SoftApModeConfiguration) msg.obj), true);
                            transitionTo(mQcApStaEnablingState);
                        } else {
                            log("StaEnabledState:CMD_RESTART_WIFI_CONTINUE: SoftApConfig obj null. Do nothing");
                        }
                        mRestartStaSapStack = false;
                    }
                    break;
                default:
                    return NOT_HANDLED;

            }
            return HANDLED;
        }
    }

// Wifi状态机启动supplicant
frameworks/opt/net/wifi/service/java/com/android/server/wifi/WifiStateMachine.java

    public void setSupplicantRunning(boolean enable)

        sendMessage(CMD_START_SUPPLICANT);


WifiStateMachine在WifiInjector中创建
	wifi状态机启动wpa_supplicant
	
frameworks/opt/net/wifi/service/java/com/android/server/wifi/WifiStateMachine.java

    class InitialState extends State {

	        public boolean processMessage(Message message) {
            logStateAndMessage(message, this);
            switch (message.what) {
                case CMD_START_SUPPLICANT:
                    Pair<Integer, IClientInterface> statusAndInterface =
                            mWifiNative.setupForClientMode();
                    if (statusAndInterface.first == WifiNative.SETUP_SUCCESS) {
                        mClientInterface = statusAndInterface.second;
                    } else {
                        incrementMetricsForSetupFailure(statusAndInterface.first);
                    }
                    if (mClientInterface == null
                            || !mDeathRecipient.linkToDeath(mClientInterface.asBinder())) {
                        setWifiState(WifiManager.WIFI_STATE_UNKNOWN);
                        staCleanup();
                        break;
                    }
					
					
frameworks/opt/net/wifi/service/java/com/android/server/wifi/WifiNative.java

   public Pair<Integer, IClientInterface> setupForClientMode() {
        if (!startHalIfNecessary(true)) {
            Log.e(mTAG, "Failed to start HAL for client mode");
            return Pair.create(SETUP_FAILURE_HAL, null);
        }
        IClientInterface iClientInterface = mWificondControl.setupDriverForClientMode();
        if (iClientInterface == null) {
            return Pair.create(SETUP_FAILURE_WIFICOND, null);
        }
        return Pair.create(SETUP_SUCCESS, iClientInterface);
    }
	
	
	
	private boolean startHalIfNecessary(boolean isStaMode) {
        if (!mWifiVendorHal.isVendorHalSupported()) {
            Log.i(mTAG, "Vendor HAL not supported, Ignore start...");
            return true;
        }
        if (mStaAndAPConcurrency)
            return mWifiVendorHal.startConcurrentVendorHal(isStaMode);

        return mWifiVendorHal.startVendorHal(isStaMode);
    }
	
	
	
	
	
	   public boolean startVendorHal(boolean isStaMode) {
        synchronized (sLock) {
            if (mIWifiStaIface != null) return boolResult(false);
            if (mIWifiApIface != null) return boolResult(false);
            if (!mHalDeviceManager.start()) {
                return startFailedTo("start the vendor HAL");
            }
            IWifiIface iface;
            if (isStaMode) {
                mIWifiStaIface = mHalDeviceManager.createStaIface(null, null);
                if (mIWifiStaIface == null) {
                    return startFailedTo("create STA Iface");
                }
                iface = (IWifiIface) mIWifiStaIface;
                if (!registerStaIfaceCallback()) {
                    return startFailedTo("register sta iface callback");
                }
                mIWifiRttController = mHalDeviceManager.createRttController(iface);
                if (mIWifiRttController == null) {
                    return startFailedTo("create RTT controller");
                }
                if (!registerRttEventCallback()) {
                    return startFailedTo("register RTT iface callback");
                }
                enableLinkLayerStats();
            } else {
                mIWifiApIface = mHalDeviceManager.createApIface(null, null);
                if (mIWifiApIface == null) {
                    return startFailedTo("create AP Iface");
                }
                iface = (IWifiIface) mIWifiApIface;
            }
            mIWifiChip = mHalDeviceManager.getChip(iface);
            if (mIWifiChip == null) {
                return startFailedTo("get the chip created for the Iface");
            }
            if (!registerChipCallback()) {
                return startFailedTo("register chip callback");
            }
            mLog.i("Vendor Hal started successfully");
            return true;
        }
    }
	
	
	
frameworks/opt/net/wifi/service/java/com/android/server/wifi/HalDeviceManage.java
	
	
	    private IWifiIface createIface(int ifaceType, InterfaceDestroyedListener destroyedListener,
            Looper looper) {
        if (DBG) Log.d(TAG, "createIface: ifaceType=" + ifaceType);

        synchronized (mLock) {
            WifiChipInfo[] chipInfos = getAllChipInfo();
            if (chipInfos == null) {
                Log.e(TAG, "createIface: no chip info found");
                stopWifi(); // major error: shutting down
                return null;
            }

            if (!validateInterfaceCache(chipInfos)) {
                Log.e(TAG, "createIface: local cache is invalid!");
                stopWifi(); // major error: shutting down
                return null;
            }

            IWifiIface iface = createIfaceIfPossible(chipInfos, ifaceType, destroyedListener,
                    looper);
            if (iface != null) { // means that some configuration has changed
                if (!dispatchAvailableForRequestListeners()) {
                    return null; // catastrophic failure - shut down
                }
            }

            return iface;
        }
		
		
		
		
		  private IWifiIface createIfaceIfPossible(WifiChipInfo[] chipInfos, int ifaceType,
            InterfaceDestroyedListener destroyedListener, Looper looper) {
			
			
			
			
			                IWifiIface iface = executeChipReconfiguration(bestIfaceCreationProposal, ifaceType);

							
							
							
							
							
							                    WifiStatus status = ifaceCreationData.chipInfo.chip.configureChip(
                            ifaceCreationData.chipModeId);
							
							
							
							
hardware\interfaces\wifi\1.1\default\wifi_chip.cpp

Return<void> WifiChip::configureChip(ChipModeId mode_id,
                                     configureChip_cb hidl_status_cb) {
  return validateAndCall(this,
                         WifiStatusCode::ERROR_WIFI_CHIP_INVALID,
                         &WifiChip::configureChipInternal,
                         hidl_status_cb,
                         mode_id);
}


WifiStatus WifiChip::configureChipInternal(ChipModeId mode_id) {
  if (mode_id != kStaChipModeId && mode_id != kApChipModeId) {
    return createWifiStatus(WifiStatusCode::ERROR_INVALID_ARGS);
  }
  if (mode_id == current_mode_id_) {
    LOG(DEBUG) << "Already in the specified mode " << mode_id;
    return createWifiStatus(WifiStatusCode::SUCCESS);
  }
  WifiStatus status = handleChipConfiguration(mode_id);
  if (status.code != WifiStatusCode::SUCCESS) {
    for (const auto& callback : event_cb_handler_.getCallbacks()) {
      if (!callback->onChipReconfigureFailure(status).isOk()) {
        LOG(ERROR) << "Failed to invoke onChipReconfigureFailure callback";

      }
 
 }
    return status;
  }
  for (const auto& callback : event_cb_handler_.getCallbacks()) {
    if (!callback->onChipReconfigured(mode_id).isOk()) {
      LOG(ERROR) << "Failed to invoke onChipReconfigured callback";
    }
  }
  current_mode_id_ = mode_id;
  return status;
}

hardware\interfaces\wifi\1.1\default\wifi_mode_controller.cpp

WifiStatus WifiChip::handleChipConfiguration(ChipModeId mode_id) {
  // If the chip is already configured in a different mode, stop
  // the legacy HAL and then start it after firmware mode change.
  // Currently the underlying implementation has a deadlock issue.
  // We should return ERROR_NOT_SUPPORTED if chip is already configured in
  // a different mode.

  // handleChipConfiguration assumes that initial mode is kInvalidModeId.
  // This limitation is not applicable for wifiStaSapConcurrency feature.
  if (!(WifiFeatureFlags::wifiStaSapConcurrency) &&
      current_mode_id_ != kInvalidModeId) {
    // TODO(b/37446050): Fix the deadlock.
    return createWifiStatus(WifiStatusCode::ERROR_NOT_SUPPORTED);
  }
  bool success;
  if (mode_id == kStaChipModeId) {
    success = mode_controller_.lock()->changeFirmwareMode(IfaceType::STA);
  } else {
    success = mode_controller_.lock()->changeFirmwareMode(IfaceType::AP);
  }
  if (!success) {
    return createWifiStatus(WifiStatusCode::ERROR_UNKNOWN);
  }
  legacy_hal::wifi_error legacy_status = legacy_hal_.lock()->start();
  if (legacy_status != legacy_hal::WIFI_SUCCESS) {
    LOG(ERROR) << "Failed to start legacy HAL: "
               << legacyErrorToString(legacy_status);
    return createWifiStatusFromLegacyError(legacy_status);
  }
  return createWifiStatus(WifiStatusCode::SUCCESS);
}

bool WifiModeController::changeFirmwareMode(IfaceType type) {
  if (!driver_tool_->LoadDriver()) {				// 加载驱动
    LOG(ERROR) << "Failed to load WiFi driver";
    return false;
  }
  if (!driver_tool_->ChangeFirmwareMode(convertIfaceTypeToFirmwareMode(type))) {
    LOG(ERROR) << "Failed to change firmware mode";
    return false;
  }
  return true;
}

frameworks\opt\net\wifi\libwifi_hal\driver_tool.cpp

frameworks\opt\net\wifi\libwifi_hal\driver_tool.cpp
bool DriverTool::LoadDriver() {
  return ::wifi_load_driver() == 0;
}


frameworks\opt\net\wifi\libwifi_hal\wifi_hal_common.cpp
int wifi_load_driver() {
#ifdef WIFI_DRIVER_MODULE_PATH
  if (is_wifi_driver_loaded()) {
    return 0;
  }

  if (insmod(DRIVER_MODULE_PATH, DRIVER_MODULE_ARG) < 0) return -1;
#endif

#ifdef WIFI_DRIVER_STATE_CTRL_PARAM
  if (is_wifi_driver_loaded()) {
    return 0;
  }

  if (wifi_change_driver_state(WIFI_DRIVER_STATE_ON) < 0) return -1;
#endif
  property_set(DRIVER_PROP_NAME, "ok");
  return 0;
}

```