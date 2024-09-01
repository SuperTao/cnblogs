查看一下android wifi扫描的过程。
```
packages\apps\Settings\src\com\android\settings\wifi\WifiSettings.java
   public void onStart() {
        super.onStart();

        // On/off switch is hidden for Setup Wizard (returns null)
        mWifiEnabler = createWifiEnabler();

        mWifiTracker.startTracking();

        if (mIsRestricted) {
            restrictUi();
            return;
        }

        onWifiStateChanged(mWifiManager.getWifiState());
    }
	
	frameworks\base\packages\SettingsLib\src\com\android\settingslib\wifi\WifiTracker.java
	    public void startTracking() {
        synchronized (mLock) {
            registerScoreCache();
            // 是否显示wifi评分的UI
            mNetworkScoringUiEnabled =
                    Settings.Global.getInt(
                            mContext.getContentResolver(),
                            Settings.Global.NETWORK_SCORING_UI_ENABLED, 0) == 1;
            // 是否显示wifi速度
            mMaxSpeedLabelScoreCacheAge =
                    Settings.Global.getLong(
                            mContext.getContentResolver(),
                            Settings.Global.SPEED_LABEL_CACHE_EVICTION_AGE_MILLIS,
                            DEFAULT_MAX_CACHED_SCORE_AGE_MILLIS);

            resumeScanning();
            if (!mRegistered) {
                mContext.registerReceiver(mReceiver, mFilter);
                // NetworkCallback objects cannot be reused. http://b/20701525 .
                mNetworkCallback = new WifiTrackerNetworkCallback();
                mConnectivityManager.registerNetworkCallback(mNetworkRequest, mNetworkCallback);
                mRegistered = true;
            }
        }

    }
	扫描服务
	    public void resumeScanning() {
        if (mScanner == null) {
            mScanner = new Scanner();
        }

        mWorkHandler.sendEmptyMessage(WorkHandler.MSG_RESUME);
        if (mWifiManager.isWifiEnabled()) {
            mScanner.resume();		// 发送扫描消息
        }
    }
	
	// 
	    class Scanner extends Handler {
        static final int MSG_SCAN = 0;

        private int mRetry = 0;

        void resume() {
            if (!hasMessages(MSG_SCAN)) {
                sendEmptyMessage(MSG_SCAN);
            }
        }

        void forceScan() {
            removeMessages(MSG_SCAN);
            sendEmptyMessage(MSG_SCAN);
        }

        void pause() {
            mRetry = 0;
            removeMessages(MSG_SCAN);
        }

        @VisibleForTesting
        boolean isScanning() {
            return hasMessages(MSG_SCAN);
        }

        @Override
        public void handleMessage(Message message) {
            if (message.what != MSG_SCAN) return;
            if (mWifiManager.startScan()) {				// 启动扫描
                mRetry = 0;
            } else if (++mRetry >= 3) {
                mRetry = 0;
                if (mContext != null) {
                    Toast.makeText(mContext, R.string.wifi_fail_to_scan, Toast.LENGTH_LONG).show();
                }
                return;
            }
			// 扫描界面扫描间隔。
            sendEmptyMessageDelayed(MSG_SCAN, WIFI_RESCAN_INTERVAL_MS);
        }
    }
	
	
	frameworks\base\wifi\java\android\net\wifi\WifiManager.java
	    /**
     * Request a scan for access points. Returns immediately. The availability
     * of the results is made known later by means of an asynchronous event sent
     * on completion of the scan.
     * @return {@code true} if the operation succeeded, i.e., the scan was initiated
     */
    public boolean startScan() {
        return startScan(null);
    }

    /** @hide */
    @SystemApi
    @RequiresPermission(android.Manifest.permission.UPDATE_DEVICE_STATS)
    public boolean startScan(WorkSource workSource) {
        try {
            String packageName = mContext.getOpPackageName();
            mService.startScan(null, workSource, packageName);
            return true;
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }
	
	frameworks\opt\net\wifi\service\java\com\android\server\wifi\WifiServiceImpl.java
	    /**
     * see {@link android.net.wifi.WifiManager#startScan}
     * and {@link android.net.wifi.WifiManager#startCustomizedScan}
     *
     * @param settings If null, use default parameter, i.e. full scan.
     * @param workSource If null, all blame is given to the calling uid.
     * @param packageName Package name of the app that requests wifi scan.
     */
    @Override
    public void startScan(ScanSettings settings, WorkSource workSource, String packageName) {
        enforceChangePermission();

        mLog.info("startScan uid=%").c(Binder.getCallingUid()).flush();
        // Check and throttle background apps for wifi scan.
        if (isRequestFromBackground(packageName)) {
            long lastScanMs = mLastScanTimestamps.getOrDefault(packageName, 0L);
            long elapsedRealtime = mClock.getElapsedSinceBootMillis();

            if (lastScanMs != 0 && (elapsedRealtime - lastScanMs) < mBackgroundThrottleInterval) {
                sendFailedScanBroadcast();
                return;
            }
            // Proceed with the scan request and record the time.
            mLastScanTimestamps.put(packageName, elapsedRealtime);
        }
        synchronized (this) {
            if (mWifiScanner == null) {
                mWifiScanner = mWifiInjector.getWifiScanner();
            }
            if (mInIdleMode) {
                // Need to send an immediate scan result broadcast in case the
                // caller is waiting for a result ..

                // TODO: investigate if the logic to cancel scans when idle can move to
                // WifiScanningServiceImpl.  This will 1 - clean up WifiServiceImpl and 2 -
                // avoid plumbing an awkward path to report a cancelled/failed scan.  This will
                // be sent directly until b/31398592 is fixed.
                sendFailedScanBroadcast();
                mScanPending = true;
                return;
            }
        }
        if (settings != null) {
            settings = new ScanSettings(settings);
            if (!settings.isValid()) {
                Slog.e(TAG, "invalid scan setting");
                return;
            }
        }
        if (workSource != null) {
            enforceWorkSourcePermission();
            // WifiManager currently doesn't use names, so need to clear names out of the
            // supplied WorkSource to allow future WorkSource combining.
            workSource.clearNames();
        }
        if (workSource == null && Binder.getCallingUid() >= 0) {
            workSource = new WorkSource(Binder.getCallingUid());
        }
        mWifiStateMachine.startScan(Binder.getCallingUid(), scanRequestCounter++,
                settings, workSource);
    }
	
	
	    class SupplicantStartedState extends State {
		
		                case CMD_START_SCAN:
                    // TODO: remove scan request path (b/31445200)
                    handleScanRequest(message);
                    break;
					
					
					
   private void handleScanRequest(Message message) {
        ScanSettings settings = null;
        WorkSource workSource = null;

        // unbundle parameters
        Bundle bundle = (Bundle) message.obj;

        if (bundle != null) {
            settings = bundle.getParcelable(CUSTOMIZED_SCAN_SETTING);
            workSource = bundle.getParcelable(CUSTOMIZED_SCAN_WORKSOURCE);
        }

        Set<Integer> freqs = null;
        if (settings != null && settings.channelSet != null) {
            freqs = new HashSet<>();
            for (WifiChannel channel : settings.channelSet) {
                freqs.add(channel.freqMHz);
            }
        }

        // Retrieve the list of hidden network SSIDs to scan for.
        List<WifiScanner.ScanSettings.HiddenNetwork> hiddenNetworks =
                mWifiConfigManager.retrieveHiddenNetworkList();

        // call wifi native to start the scan
        if (startScanNative(freqs, hiddenNetworks, workSource)) {
            // a full scan covers everything, clearing scan request buffer
            if (freqs == null)
                mBufferedScanMsg.clear();
            messageHandlingStatus = MESSAGE_HANDLING_STATUS_OK;
            return;
        }

        // if reach here, scan request is rejected

        if (!mIsScanOngoing) {
            // if rejection is NOT due to ongoing scan (e.g. bad scan parameters),

            // discard this request and pop up the next one
            if (mBufferedScanMsg.size() > 0) {
                sendMessage(mBufferedScanMsg.remove());
            }
            messageHandlingStatus = MESSAGE_HANDLING_STATUS_DISCARD;
        } else if (!mIsFullScanOngoing) {
            // if rejection is due to an ongoing scan, and the ongoing one is NOT a full scan,
            // buffer the scan request to make sure specified channels will be scanned eventually
            if (freqs == null)
                mBufferedScanMsg.clear();
            if (mBufferedScanMsg.size() < SCAN_REQUEST_BUFFER_MAX_SIZE) {
                Message msg = obtainMessage(CMD_START_SCAN,
                        message.arg1, message.arg2, bundle);
                mBufferedScanMsg.add(msg);
            } else {
                // if too many requests in buffer, combine them into a single full scan
                bundle = new Bundle();
                bundle.putParcelable(CUSTOMIZED_SCAN_SETTING, null);
                bundle.putParcelable(CUSTOMIZED_SCAN_WORKSOURCE, workSource);
                Message msg = obtainMessage(CMD_START_SCAN, message.arg1, message.arg2, bundle);
                mBufferedScanMsg.clear();
                mBufferedScanMsg.add(msg);
            }
            messageHandlingStatus = MESSAGE_HANDLING_STATUS_LOOPED;
        } else {
            // mIsScanOngoing and mIsFullScanOngoing
            messageHandlingStatus = MESSAGE_HANDLING_STATUS_FAIL;
        }
    }
	
	
	
	
	   private boolean startScanNative(final Set<Integer> freqs,
            List<WifiScanner.ScanSettings.HiddenNetwork> hiddenNetworkList,
            WorkSource workSource) {
        WifiScanner.ScanSettings settings = new WifiScanner.ScanSettings();
        if (freqs == null) {
            settings.band = WifiScanner.WIFI_BAND_BOTH_WITH_DFS;
        } else {
            settings.band = WifiScanner.WIFI_BAND_UNSPECIFIED;
            int index = 0;
            settings.channels = new WifiScanner.ChannelSpec[freqs.size()];
            for (Integer freq : freqs) {
                settings.channels[index++] = new WifiScanner.ChannelSpec(freq);
            }
        }
        settings.reportEvents = WifiScanner.REPORT_EVENT_AFTER_EACH_SCAN
                | WifiScanner.REPORT_EVENT_FULL_SCAN_RESULT;

        settings.hiddenNetworks =
                hiddenNetworkList.toArray(
                        new WifiScanner.ScanSettings.HiddenNetwork[hiddenNetworkList.size()]);

        WifiScanner.ScanListener nativeScanListener = new WifiScanner.ScanListener() {
                // ignore all events since WifiStateMachine is registered for the supplicant events
                @Override
                public void onSuccess() {
                }
                @Override
                public void onFailure(int reason, String description) {
                    mIsScanOngoing = false;
                    mIsFullScanOngoing = false;
                }
                @Override
                public void onResults(WifiScanner.ScanData[] results) {
                }
                @Override
                public void onFullResult(ScanResult fullScanResult) {
                }
                @Override
                public void onPeriodChanged(int periodInMs) {
                }
            };
        mWifiScanner.startScan(settings, nativeScanListener, workSource);
        mIsScanOngoing = true;
        mIsFullScanOngoing = (freqs == null);
        lastScanFreqs = freqs;
        return true;
    }
	
	
	
	frameworks\opt\net\wifi\service\java\com\android\server\wifi\scanner\WifiScanningServiceImpl.java
	   class DriverStartedState extends State {
            @Override
            public void exit() {
                // clear scan results when scan mode is not active
                mCachedScanResults.clear();

                mWifiMetrics.incrementScanReturnEntry(
                        WifiMetricsProto.WifiLog.SCAN_FAILURE_INTERRUPTED,
                        mPendingScans.size());
                sendOpFailedToAllAndClear(mPendingScans, WifiScanner.REASON_UNSPECIFIED,
                        "Scan was interrupted");
            }

            @Override
            public boolean processMessage(Message msg) {
                ClientInfo ci = mClients.get(msg.replyTo);

                switch (msg.what) {
                    case WifiScanner.CMD_START_SINGLE_SCAN:
                        mWifiMetrics.incrementOneshotScanCount();
                        int handler = msg.arg2;
                        Bundle scanParams = (Bundle) msg.obj;
                        if (scanParams == null) {
                            logCallback("singleScanInvalidRequest",  ci, handler, "null params");
                            replyFailed(msg, WifiScanner.REASON_INVALID_REQUEST, "params null");
                            return HANDLED;
                        }
                        scanParams.setDefusable(true);
                        ScanSettings scanSettings =
                                scanParams.getParcelable(WifiScanner.SCAN_PARAMS_SCAN_SETTINGS_KEY);
                        WorkSource workSource =
                                scanParams.getParcelable(WifiScanner.SCAN_PARAMS_WORK_SOURCE_KEY);
                        if (validateScanRequest(ci, handler, scanSettings, workSource)) {
                            logScanRequest("addSingleScanRequest", ci, handler, workSource,
                                    scanSettings, null);
                            replySucceeded(msg);

                            // If there is an active scan that will fulfill the scan request then
                            // mark this request as an active scan, otherwise mark it pending.
                            // If were not currently scanning then try to start a scan. Otherwise
                            // this scan will be scheduled when transitioning back to IdleState
                            // after finishing the current scan.
                            if (getCurrentState() == mScanningState) {
                                if (activeScanSatisfies(scanSettings)) {
                                    mActiveScans.addRequest(ci, handler, workSource, scanSettings);
                                } else {
                                    mPendingScans.addRequest(ci, handler, workSource, scanSettings);
                                }
                            } else {
                                mPendingScans.addRequest(ci, handler, workSource, scanSettings);
                                tryToStartNewScan();
                            }
                        } else {
                            logCallback("singleScanInvalidRequest",  ci, handler, "bad request");
                            replyFailed(msg, WifiScanner.REASON_INVALID_REQUEST, "bad request");
                            mWifiMetrics.incrementScanReturnEntry(
                                    WifiMetricsProto.WifiLog.SCAN_FAILURE_INVALID_CONFIGURATION, 1);
                        }
                        return HANDLED;
                    case WifiScanner.CMD_STOP_SINGLE_SCAN:
                        removeSingleScanRequest(ci, msg.arg2);
                        return HANDLED;
                    default:
                        return NOT_HANDLED;
                }
            }
        }
		
		
	frameworks\opt\net\wifi\service\java\com\android\server\wifi\scanner\WificondScannerImpl.java
	
    public boolean startSingleScan(WifiNative.ScanSettings settings,
            WifiNative.ScanEventHandler eventHandler) {
        if (eventHandler == null || settings == null) {
            Log.w(TAG, "Invalid arguments for startSingleScan: settings=" + settings
                    + ",eventHandler=" + eventHandler);
            return false;
        }
        if (mPendingSingleScanSettings != null
                || (mLastScanSettings != null && mLastScanSettings.singleScanActive)) {
            Log.w(TAG, "A single scan is already running");
            return false;
        }
        synchronized (mSettingsLock) {
            mPendingSingleScanSettings = settings;
            mPendingSingleScanEventHandler = eventHandler;
            processPendingScans();
            return true;
        }
    }
```