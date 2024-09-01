记录android 8 蓝牙扫描设备的流程
```

src/com/android/settings/bluetooth/BluetoothSettings.java
    @Override
    protected List<AbstractPreferenceController> getPreferenceControllers(Context context) {
        final List<AbstractPreferenceController> controllers = new ArrayList<>();
        final Lifecycle lifecycle = getLifecycle();
        mDeviceNamePrefController = new BluetoothDeviceNamePreferenceController(context, lifecycle);
        mPairingPrefController = new BluetoothPairingPreferenceController(context, this,
                (SettingsActivity) getActivity());
        controllers.add(mDeviceNamePrefController);
        controllers.add(mPairingPrefController);
        controllers.add(new BluetoothFilesPreferenceController(context));
        controllers.add(new BluetoothDeviceRenamePreferenceController(context, this, lifecycle));

        return controllers;
    }  

src/com/android/settings/bluetooth/BluetoothPairingPreferenceController.java
    public boolean handlePreferenceTreeClick(Preference preference) {
        if (KEY_PAIRING.equals(preference.getKey())) {
            mActivity.startPreferencePanelAsUser(mFragment, BluetoothPairingDetail.class.getName(),
                    null, R.string.bluetooth_pairing_page_title, null,
                    new UserHandle(UserHandle.myUserId()));
            return true;
        }   

        return false;
    }

src/com/android/settings/bluetooth/BluetoothPairingDetail.java
    @VisibleForTesting
    void updateContent(int bluetoothState) {
        switch (bluetoothState) {
            case BluetoothAdapter.STATE_ON:
                mDevicePreferenceMap.clear();
                mLocalAdapter.setBluetoothEnabled(true);

                addDeviceCategory(mAvailableDevicesCategory,
                        R.string.bluetooth_preference_found_devices,
                        BluetoothDeviceFilter.UNBONDED_DEVICE_FILTER, mInitialScanStarted);
                updateFooterPreference(mFooterPreference);
                mAlwaysDiscoverable.start();                            
                enableScanning(); 		// 扫描                                                      
                break;
                                                                                                        
            case BluetoothAdapter.STATE_OFF:
                finish();                                                                                                               
                break;
        }                                                                                                                                        
    } 
	
	
	@Override
    void enableScanning() {
        // Clear all device states before first scan
        if (!mInitialScanStarted) {
            if (mAvailableDevicesCategory != null) {
                removeAllDevices();
            }
            mLocalManager.getCachedDeviceManager().clearNonBondedDevices();
            mInitialScanStarted = true;
        }
        super.enableScanning();
    }
	
packages/apps/Settings/src/com/android/settings/bluetooth/DeviceListPreferenceFragment.java 
	@VisibleForTesting
    void enableScanning() {
        // LocalBluetoothAdapter already handles repeated scan requests
        mLocalAdapter.startScanning(true);
        mScanEnabled = true;
    }
	
	
frameworks\base\packages\SettingsLib\src\com\android\settingslib\bluetooth\LocalBluetoothAdapter.java
	    public void startScanning(boolean force) {
        // Only start if we're not already scanning
        if (!mAdapter.isDiscovering()) {		// 判断是否正在扫描
            if (!force) {
                // Don't scan more than frequently than SCAN_EXPIRATION_MS,
                // unless forced
                if (mLastScan + SCAN_EXPIRATION_MS > System.currentTimeMillis()) {
                    return;
                }
				// 放音乐的时候不进行扫描，除非强制
                // If we are playing music, don't scan unless forced.
                A2dpProfile a2dp = mProfileManager.getA2dpProfile();
                if (a2dp != null && a2dp.isA2dpPlaying()) {
                    return;
                }
                A2dpSinkProfile a2dpSink = mProfileManager.getA2dpSinkProfile();
                if ((a2dpSink != null) && (a2dpSink.isA2dpPlaying())){
                    return;
                }
            }

            if (mAdapter.startDiscovery()) {
                mLastScan = System.currentTimeMillis();
            }
        }
    }

frameworks\base\core\java\android\bluetooth\BluetoothAdapter.java
    public boolean startDiscovery() {
        android.util.SeempLog.record(58);
        if (getState() != STATE_ON) return false;
        try {
            mServiceLock.readLock().lock();
            if (mService != null) return mService.startDiscovery();
        } catch (RemoteException e) {
            Log.e(TAG, "", e);
        } finally {
            mServiceLock.readLock().unlock();
        }
        return false;
    }
	
	
packages\apps\Bluetooth\src\com\android\bluetooth\btservice\AdapterService.java

    private static class AdapterServiceBinder extends IBluetooth.Stub {

	
	       public boolean startDiscovery() {
            if (!Utils.checkCaller()) {
                Log.w(TAG, "startDiscovery() - Not allowed for non-active user");
                return false;
            }

            AdapterService service = getService();
            if (service == null) return false;
            return service.startDiscovery();
        }
		
     boolean startDiscovery() {
        debugLog("startDiscovery");
        enforceCallingOrSelfPermission(BLUETOOTH_ADMIN_PERM,
                                       "Need BLUETOOTH ADMIN permission");
        //do not allow new connections with active multicast
        A2dpService a2dpService = A2dpService.getA2dpService();
        if (a2dpService != null &&
            a2dpService.isMulticastFeatureEnabled() &&
            a2dpService.isMulticastOngoing(null)) {
            Log.i(TAG,"A2dp Multicast is Ongoing, ignore discovery");
            return false;
        }

        if (mAdapterProperties.isDiscovering()) {
            Log.i(TAG,"discovery already active, ignore startDiscovery");
            return false;
        }
        return startDiscoveryNative();
    }
```
Liu Tao 
2019-3-27