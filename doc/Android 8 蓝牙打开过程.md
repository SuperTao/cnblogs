```
packages\apps\Settings\src\com\android\settings\bluetooth\BluetoothEnabler.java

    @Override
    public boolean onSwitchToggled(boolean isChecked) {
        if (maybeEnforceRestrictions()) {
            return true;
        }
        // 显示toast，如果飞行模式不允许蓝牙打开
        // Show toast message if Bluetooth is not allowed in airplane mode
        if (isChecked &&
                !WirelessUtils.isRadioAllowed(mContext, Settings.Global.RADIO_BLUETOOTH)) {
            Toast.makeText(mContext, R.string.wifi_in_airplane_mode, Toast.LENGTH_SHORT).show();
            // Reset switch to off
            mSwitch.setChecked(false);
            return false;
        }

        mMetricsFeatureProvider.action(mContext, mMetricsEvent, isChecked);

        if (mLocalAdapter != null) {
            // 打开或者关闭蓝牙
            boolean status = mLocalAdapter.setBluetoothEnabled(isChecked);
            // If we cannot toggle it ON then reset the UI assets:
            // a) The switch should be OFF but it should still be togglable (enabled = True)
            // b) The switch bar should have OFF text.
            if (isChecked && !status) {
                mSwitch.setChecked(false);
                mSwitch.setEnabled(true);
                mSwitchWidget.updateTitle(false);
                return false;
            }
        }
        // 切换蓝牙开关switch的状态
        mSwitchWidget.setEnabled(false);
        return true;
    }
	
	
	
frameworks\base\packages\SettingsLib\src\com\android\settingslib\bluetooth\LocalBluetoothAdapter.java
    public boolean setBluetoothEnabled(boolean enabled) {
        // 打开或者关闭蓝牙
        boolean success = enabled
                ? mAdapter.enable()
                : mAdapter.disable();

        if (success) {
            setBluetoothStateInt(enabled
                ? BluetoothAdapter.STATE_TURNING_ON
                : BluetoothAdapter.STATE_TURNING_OFF);
        } else {
            if (Utils.V) {
                Log.v(TAG, "setBluetoothEnabled call, manager didn't return " +
                        "success for enabled: " + enabled);
            }

            syncBluetoothState();
        }
        return success;
    }



frameworks\base\core\java\android\bluetooth\BluetoothAdapter.java
    @RequiresPermission(Manifest.permission.BLUETOOTH_ADMIN)
    public boolean enable() {
        android.util.SeempLog.record(56);
        if (isEnabled()) {
            if (DBG) Log.d(TAG, "enable(): BT already enabled!");
            return true;
        }
        try {
            // 打开蓝牙
            return mManagerService.enable(ActivityThread.currentPackageName());
        } catch (RemoteException e) {Log.e(TAG, "", e);}
        return false;
    }	
	

	
frameworks\base\services\core\java\com\android\server\BluetoothManagerService.java	
	    public boolean enable(String packageName) throws RemoteException {
        final int callingUid = Binder.getCallingUid();
        final boolean callerSystem = UserHandle.getAppId(callingUid) == Process.SYSTEM_UID;

        if (isBluetoothDisallowed()) {
            if (DBG) {
                Slog.d(TAG,"enable(): not enabling - bluetooth disallowed");
            }
            return false;
        }

        if (!callerSystem) {
            if (!checkIfCallerIsForegroundUser()) {
                Slog.w(TAG, "enable(): not allowed for non-active and non system user");
                return false;
            }
            // 检查是否具有操作蓝牙的权限
            mContext.enforceCallingOrSelfPermission(BLUETOOTH_ADMIN_PERM,
                    "Need BLUETOOTH ADMIN permission");

            if (!isEnabled() && mPermissionReviewRequired
                    && startConsentUiIfNeeded(packageName, callingUid,
                            BluetoothAdapter.ACTION_REQUEST_ENABLE)) {
                return false;
            }
        }
        if (isStrictOpEnable()) {
            AppOpsManager mAppOpsManager = mContext.getSystemService(AppOpsManager.class);
            String packages = mContext.getPackageManager().getNameForUid(Binder.getCallingUid());
            if ((Binder.getCallingUid() >= Process.FIRST_APPLICATION_UID)
                    && (packages.indexOf("android.uid.systemui") != 0)
                    && (packages.indexOf("android.uid.system") != 0)) {
                int result = mAppOpsManager.noteOp(AppOpsManager.OP_BLUETOOTH_ADMIN,
                        Binder.getCallingUid(), packages);
                if (result == AppOpsManager.MODE_IGNORED) {
                    return false;
                }
            }
        }
        if (DBG) {
            Slog.d(TAG,"enable(" + packageName + "):  mBluetooth =" + mBluetooth +
                    " mBinding = " + mBinding + " mState = " +
                    BluetoothAdapter.nameForState(mState));
        }

        synchronized(mReceiver) {
            mQuietEnableExternal = false;
            mEnableExternal = true;
            // waive WRITE_SECURE_SETTINGS permission check
            // 发送消息
            sendEnableMsg(false, packageName);
        }
        if (DBG) Slog.d(TAG, "enable returning");
        return true;
    }
	
	
	
	    private void sendEnableMsg(boolean quietMode, String packageName) {
        mHandler.sendMessage(mHandler.obtainMessage(MESSAGE_ENABLE,
                             quietMode ? 1 : 0, 0));
        addActiveLog(packageName, true);
    }
	
                case MESSAGE_ENABLE:
                    if (DBG) {
                        Slog.d(TAG, "MESSAGE_ENABLE(" + msg.arg1 + "): mBluetooth = " + mBluetooth);
                    }
                    mHandler.removeMessages(MESSAGE_RESTART_BLUETOOTH_SERVICE);
                    mEnable = true;

                    // Use service interface to get the exact state
                    try {
                        mBluetoothLock.readLock().lock();
                        if (mBluetooth != null) {
                            int state = mBluetooth.getState();
                            if (state == BluetoothAdapter.STATE_BLE_ON) {
                                Slog.w(TAG, "BT Enable in BLE_ON State, going to ON");
                                mBluetooth.onLeServiceUp();
                                persistBluetoothSetting(BLUETOOTH_ON_BLUETOOTH);
                                break;
                            }
                        }
                    } catch (RemoteException e) {
                        Slog.e(TAG, "", e);
                    } finally {
                        mBluetoothLock.readLock().unlock();
                    }

                    mQuietEnable = (msg.arg1 == 1);
                    if (mBluetooth == null) {
                        // 处理打开状态
                        handleEnable(mQuietEnable);
                    } else {
                        //
                        // We need to wait until transitioned to STATE_OFF and
                        // the previous Bluetooth process has exited. The
                        // waiting period has three components:
                        // (a) Wait until the local state is STATE_OFF. This
                        //     is accomplished by "waitForMonitoredOnOff(false, true)".
                        // (b) Wait until the STATE_OFF state is updated to
                        //     all components.
                        // (c) Wait until the Bluetooth process exits, and
                        //     ActivityManager detects it.
                        // The waiting for (b) and (c) is accomplished by
                        // delaying the MESSAGE_RESTART_BLUETOOTH_SERVICE
                        // message. On slower devices, that delay needs to be
                        // on the order of (2 * SERVICE_RESTART_TIME_MS).
                        //
                        // Wait for (a) is required only when Bluetooth is being
                        // turned off.
                        int state;
                        try {
                            state = mBluetooth.getState();
                        } catch (RemoteException e) {
                            Slog.e(TAG, "getState()", e);
                            break;
                        }
                        if(state == BluetoothAdapter.STATE_TURNING_OFF || state == BluetoothAdapter.STATE_BLE_TURNING_OFF)
                            waitForMonitoredOnOff(false, true);
                        Message restartMsg = mHandler.obtainMessage(
                                MESSAGE_RESTART_BLUETOOTH_SERVICE);
                        mHandler.sendMessageDelayed(restartMsg,
                                2 * SERVICE_RESTART_TIME_MS);
                    }
                    break;
					
					
    private void handleEnable(boolean quietMode) {
        mQuietEnable = quietMode;

        try {
            mBluetoothLock.writeLock().lock();
            if ((mBluetooth == null) && (!mBinding)) {
                //Start bind timeout and bind
                Message timeoutMsg=mHandler.obtainMessage(MESSAGE_TIMEOUT_BIND);
                // bind 超时时间
                mHandler.sendMessageDelayed(timeoutMsg,TIMEOUT_BIND_MS);
                Intent i = new Intent(IBluetooth.class.getName());
                if (!doBind(i, mConnection,Context.BIND_AUTO_CREATE | Context.BIND_IMPORTANT,
                        UserHandle.CURRENT)) {
                    mHandler.removeMessages(MESSAGE_TIMEOUT_BIND);
                } else {
                    mBinding = true;
                }
            } else if (mBluetooth != null) {
                //Enable bluetooth
                try {
                    if (!mQuietEnable) {
                        // 打开蓝牙
                        if(!mBluetooth.enable()) {
                            Slog.e(TAG,"IBluetooth.enable() returned false");
                        }
                    }
                    else {
                        if(!mBluetooth.enableNoAutoConnect()) {
                            Slog.e(TAG,"IBluetooth.enableNoAutoConnect() returned false");
                        }
                    }
                } catch (RemoteException e) {
                    Slog.e(TAG,"Unable to call enable()",e);
                }
            }
        } finally {
            mBluetoothLock.writeLock().unlock();
        }
    }
	
	
packages\apps\Bluetooth\src\com\android\bluetooth\btservice\AdapterService.java
    private static class AdapterServiceBinder extends IBluetooth.Stub {
	        public boolean enable() {
            if ((Binder.getCallingUid() != Process.SYSTEM_UID) &&
                (!Utils.checkCaller())) {
                Log.w(TAG, "enable() - Not allowed for non-active user and non system user");
                return false;
            }
            AdapterService service = getService();
            if (service == null) return false;
            return service.enable();
        }

     public synchronized boolean enable(boolean quietMode) {
         enforceCallingOrSelfPermission(BLUETOOTH_ADMIN_PERM, "Need BLUETOOTH ADMIN permission");

         // Enforce the user restriction for disallowing Bluetooth if it was set.
         if (mUserManager.hasUserRestriction(UserManager.DISALLOW_BLUETOOTH, UserHandle.SYSTEM)) {
            debugLog("enable() called when Bluetooth was disallowed");
            return false;
         }

         debugLog("enable() - Enable called with quiet mode status =  " + mQuietmode);
         mQuietmode = quietMode;
         Message m = mAdapterStateMachine.obtainMessage(AdapterState.BLE_TURN_ON);
         // 发送消息
         mAdapterStateMachine.sendMessage(m);
         mBluetoothStartTime = System.currentTimeMillis();
         return true;
     }
	 
	
packages\apps\Bluetooth\src\com\android\bluetooth\btservice\AdapterState.java
	
            switch(msg.what) {
               case BLE_TURN_ON:
                   notifyAdapterStateChange(BluetoothAdapter.STATE_BLE_TURNING_ON);
                   mPendingCommandState.setBleTurningOn(true);
                   transitionTo(mPendingCommandState);
                   sendMessageDelayed(BLE_START_TIMEOUT, BLE_START_TIMEOUT_DELAY);
                   adapterService.BleOnProcessStart();
                   break;
				   
packages\apps\Bluetooth\src\com\android\bluetooth\btservice\AdapterService.java			   
    void BleOnProcessStart() {
        debugLog("BleOnProcessStart()");

        if (getResources().getBoolean(
                R.bool.config_bluetooth_reload_supported_profiles_when_enabled)) {
            Config.init(getApplicationContext());
        }

        Class[] supportedProfileServices = Config.getSupportedProfiles();
        //Initialize data objects
        for (int i=0; i < supportedProfileServices.length;i++) {
            mProfileServicesState.put(supportedProfileServices[i].getName(),BluetoothAdapter.STATE_OFF);
        }

        debugLog("BleOnProcessStart() - Make Bond State Machine");

        mJniCallbacks.init(mBondStateMachine,mRemoteDevices);

        try {
            mBatteryStats.noteResetBleScan();
        } catch (RemoteException e) {
            // Ignore.
        }
        // 启动蓝牙
        //Start Gatt service
        setGattProfileServiceState(supportedProfileServices,BluetoothAdapter.STATE_ON);
    }
```