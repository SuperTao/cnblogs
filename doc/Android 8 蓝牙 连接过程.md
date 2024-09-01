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
	
	
1. 已经连接过的设备，直接断开
	    // Show disconnect confirmation dialog for a device.
    private void askDisconnect() {
        Context context = getContext();
        String name = mCachedDevice.getName();
        if (TextUtils.isEmpty(name)) {
            name = context.getString(R.string.bluetooth_device);
        }
        String message = context.getString(R.string.bluetooth_disconnect_all_profiles, name);
        String title = context.getString(R.string.bluetooth_disconnect_title);
		// 对话框的listener
        DialogInterface.OnClickListener disconnectListener = new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialog, int which) {
                mCachedDevice.disconnect();
            }
        };
		// 显示断开对话框
        mDisconnectDialog = Utils.showDisconnectDialog(context,
                mDisconnectDialog, disconnectListener, title, Html.fromHtml(message));
    }
	
2. 匹配过的设别，进行连接
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
                    connectInt(profile);                        // 连接对应的profile
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
	
3. 没有配对的设备，先进行配对
	   private void pair() {
        if (!mCachedDevice.startPairing()) {
            Utils.showError(getContext(), mCachedDevice.getName(),
                    R.string.bluetooth_pairing_error_message);
        }
    }
	
	frameworks\base\packages\SettingsLib\src\com\android\settingslib\bluetooth\CachedBluetoothDevice.java
	   public boolean startPairing() {
        // Pairing is unreliable while scanning, so cancel discovery
        // 配对的时候，关闭扫描
		if (mLocalAdapter.isDiscovering()) {
            mLocalAdapter.cancelDiscovery();
        }

        if (!mDevice.createBond()) {
            return false;
        }

        return true;
    }
	
	frameworks\base\core\java\android\bluetooth\BluetoothDevice.java
	    @RequiresPermission(Manifest.permission.BLUETOOTH_ADMIN)
    public boolean createBond() {
        final IBluetooth service = sService;
        if (service == null) {
            Log.e(TAG, "BT not enabled. Cannot create bond to Remote Device");
            return false;
        }
        try {
            Log.i(TAG, "createBond() for device " + getAddress()
                    + " called by pid: " + Process.myPid()
                    + " tid: " + Process.myTid());
            return service.createBond(this, TRANSPORT_AUTO);
        } catch (RemoteException e) {
            Log.e(TAG, "", e);
        }
        return false;
    }
	
	packages\apps\Bluetooth\src\com\android\bluetooth\btservice\AdapterService.java
	    private static class AdapterServiceBinder extends IBluetooth.Stub {
	        public boolean createBond(BluetoothDevice device, int transport) {
            if (!Utils.checkCallerAllowManagedProfiles(mService)) {
                Log.w(TAG, "createBond() - Not allowed for non-active user");
                return false;
            }

            AdapterService service = getService();
            if (service == null) return false;
            return service.createBond(device, transport, null);		// bond
        }
		
		
		// 建立bond
	     boolean createBond(BluetoothDevice device, int transport, OobData oobData) {
        enforceCallingOrSelfPermission(BLUETOOTH_ADMIN_PERM,
            "Need BLUETOOTH ADMIN permission");
        DeviceProperties deviceProp = mRemoteDevices.getDeviceProperties(device);
        if (deviceProp != null && deviceProp.getBondState() != BluetoothDevice.BOND_NONE) {
            return false;
        }
		// 在多播的时候不要进行bond，不明白这个multicast是什么意思
        // Multicast: Do not allow bonding while multcast
        A2dpService a2dpService = A2dpService.getA2dpService();
        if (a2dpService != null &&
            a2dpService.isMulticastFeatureEnabled() &&
            a2dpService.isMulticastOngoing(null)) {
            Log.i(TAG,"A2dp Multicast is ongoing, ignore bonding");
            return false;
        }

        mRemoteDevices.setBondingInitiatedLocally(Utils.getByteAddress(device));

        // Pairing is unreliable while scanning, so cancel discovery
        // Note, remove this when native stack improves
        if (!mAdapterProperties.isDiscovering()) {
            Log.i(TAG,"discovery not active, no need to send cancelDiscovery");
        } else {
            cancelDiscoveryNative();
        }
		// 或去状态机的消息
        Message msg = mBondStateMachine.obtainMessage(BondStateMachine.CREATE_BOND);
        msg.obj = device;
        msg.arg1 = transport;

        if (oobData != null) {
            Bundle oobDataBundle = new Bundle();
            oobDataBundle.putParcelable(BondStateMachine.OOBDATA, oobData);
            msg.setData(oobDataBundle);
        }
		// 发送消息给状态机
        mBondStateMachine.sendMessage(msg);
        return true;
    }
		
packages\apps\Bluetooth\src\com\android\bluetooth\btservice\BondStateMachine.java
   private class StableState extends State {
        @Override
        public void enter() {
            infoLog("StableState(): Entering Off State");
        }

        @Override
        public synchronized boolean processMessage(Message msg) {

            BluetoothDevice dev = (BluetoothDevice)msg.obj;

            switch(msg.what) {

              case CREATE_BOND:
                  OobData oobData = null;
                  if (msg.getData() != null)
                      oobData = msg.getData().getParcelable(OOBDATA);

                  createBond(dev, msg.arg1, oobData, true);
                  break;
              case REMOVE_BOND:
                  removeBond(dev, true);
                  break;
              case BONDING_STATE_CHANGE:
                int newState = msg.arg1;
                /* if incoming pairing, transition to pending state */
                if (newState == BluetoothDevice.BOND_BONDING)
                {
                    if(!mDevices.contains(dev)) {
                        mDevices.add(dev);
                    }
                    sendIntent(dev, newState, 0);
                    transitionTo(mPendingCommandState);
                }
                else if (newState == BluetoothDevice.BOND_NONE)
                {
                    /* if the link key was deleted by the stack */
                    sendIntent(dev, newState, 0);
                }
                else
                {
                    Log.e(TAG, "In stable state, received invalid newState: " + newState);
                }
                break;

              case CANCEL_BOND:
              default:
                   Log.e(TAG, "Received unhandled state: " + msg.what);
                   return false;
            }
            return true;
        }
    }
	

   private boolean createBond(BluetoothDevice dev, int transport, OobData oobData,
                               boolean transition) {
        if(mAdapterService == null) return false;
        if (dev.getBondState() == BluetoothDevice.BOND_NONE) {
            infoLog("Bond address is:" + dev);
            byte[] addr = Utils.getBytesFromAddress(dev.getAddress());
            boolean result;
			// 调用JNI
            if (oobData != null) {
                result = mAdapterService.createBondOutOfBandNative(addr, transport, oobData);
            } else {
                result = mAdapterService.createBondNative(addr, transport);
            }

            if (!result) {
                sendIntent(dev, BluetoothDevice.BOND_NONE,
                           BluetoothDevice.UNBOND_REASON_REMOVED);
                return false;
            } else if (transition) {
                transitionTo(mPendingCommandState);
            }
            return true;
        }
        return false;
    }

```
Liu Tao 

2019-3-27