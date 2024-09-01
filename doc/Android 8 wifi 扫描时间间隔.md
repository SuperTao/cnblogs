```
wifi setting界面扫描时间间隔：10s
不在wifi setting界面，扫描时间间隔，最小20s，然后按找2倍的间隔进行递增，40s,60s..., 最大160s
PNO 即Preferred Network Offload，用于系统在休眠的时候连接WiFi 

1. wifi setting界面
packages\apps\Settings\src\com\android\settings\wifi\WifiSettings.java
   public void onStart() {
        mWifiTracker.startTracking();

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
            mScanner.resume();      // 发送扫描消息
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
            if (mWifiManager.startScan()) {             // 启动扫描
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
//private static final int WIFI_RESCAN_INTERVAL_MS = 10 * 1000;
		
2. 不在wifisetting界面，亮屏情况下。
frameworks\opt\net\wifi\service\java\com\android\server\wifi\WifiConnectivityManager.java
		
	public void handleScreenStateChanged(boolean screenOn) {
        localLog("handleScreenStateChanged: screenOn=" + screenOn);

        mScreenOn = screenOn;

        mOpenNetworkNotifier.handleScreenStateChanged(screenOn);

        startConnectivityScan(SCAN_ON_SCHEDULE);
    }
	
	    private void startConnectivityScan(boolean scanImmediately) {
        localLog("startConnectivityScan: screenOn=" + mScreenOn
                + " wifiState=" + stateToString(mWifiState)
                + " scanImmediately=" + scanImmediately
                + " wifiEnabled=" + mWifiEnabled
                + " wifiConnectivityManagerEnabled="
                + mWifiConnectivityManagerEnabled);

        if (!mWifiEnabled || !mWifiConnectivityManagerEnabled) {
            return;
        }

        // Always stop outstanding connecivity scan if there is any
        stopConnectivityScan();

        // Don't start a connectivity scan while Wifi is in the transition
        // between connected and disconnected states.
        if (mWifiState != WIFI_STATE_CONNECTED && mWifiState != WIFI_STATE_DISCONNECTED) {
            return;
        }

        if (mScreenOn) {
            startPeriodicScan(scanImmediately);
        } else {
            if (mWifiState == WIFI_STATE_DISCONNECTED && !mPnoScanStarted) {
                startDisconnectedPnoScan();
            }
        }
    }
	
	
	// Start a periodic scan when screen is on
    private void startPeriodicScan(boolean scanImmediately) {
	// 从新设置PNO扫描的时间间隔。
	// 当扫描的网络，Rssi比较低时，将会进行从新扫描
        mPnoScanListener.resetLowRssiNetworkRetryDelay();
		// 漫游关闭时，不进行扫描
        // No connectivity scan if auto roaming is disabled.
        if (mWifiState == WIFI_STATE_CONNECTED && !mEnableAutoJoinWhenAssociated) {
            return;
        }

        // Due to b/28020168, timer based single scan will be scheduled
        // to provide periodic scan in an exponential backoff fashion.
        if (scanImmediately) {
            resetLastPeriodicSingleScanTimeStamp();
        }
        mPeriodicSingleScanInterval = PERIODIC_SCAN_INTERVAL_MS;
        startPeriodicSingleScan();
    }
	
	    @VisibleForTesting
    public static final int PERIODIC_SCAN_INTERVAL_MS = 20 * 1000; // 20 seconds
	
	    // Start a single scan and set up the interval for next single scan.
    private void startPeriodicSingleScan() {
        long currentTimeStamp = mClock.getElapsedSinceBootMillis();

        if (mLastPeriodicSingleScanTimeStamp != RESET_TIME_STAMP) {
            long msSinceLastScan = currentTimeStamp - mLastPeriodicSingleScanTimeStamp;
            if (msSinceLastScan < PERIODIC_SCAN_INTERVAL_MS) {
                localLog("Last periodic single scan started " + msSinceLastScan
                        + "ms ago, defer this new scan request.");
                schedulePeriodicScanTimer(PERIODIC_SCAN_INTERVAL_MS - (int) msSinceLastScan);
                return;
            }
        }

        boolean isFullBandScan = true;
		// 判断wifi使用率是不是比较高，比较高只进行一部分扫描
		// 通过发送和接收速率判断
        // If the WiFi traffic is heavy, only partial scan is initiated.
        if (mWifiState == WIFI_STATE_CONNECTED
                && (mWifiInfo.txSuccessRate > mFullScanMaxTxRate
                    || mWifiInfo.rxSuccessRate > mFullScanMaxRxRate)) {
            localLog("No full band scan due to ongoing traffic");
            isFullBandScan = false;
        }
		// 再次判断，如果发送接收速率超过最大值，将不进行扫描。
        mLastPeriodicSingleScanTimeStamp = currentTimeStamp;
		if (mWifiState == WIFI_STATE_CONNECTED
                 && (mWifiInfo.txSuccessRate
                         > MAX_TX_PACKET_FOR_PARTIAL_SCANS
                 || mWifiInfo.rxSuccessRate
                         > MAX_RX_PACKET_FOR_PARTIAL_SCANS)) {
                         Log.e(TAG,"Ignore scan due to heavy traffic");
         } else {
                 startSingleScan(isFullBandScan, WIFI_WORK_SOURCE);
         }

        schedulePeriodicScanTimer(mPeriodicSingleScanInterval);
		// 设置下一次扫描时间间隔，是当前的2倍。
		// 最大扫描时间间隔160s
		// public static final int MAX_PERIODIC_SCAN_INTERVAL_MS = 160 * 1000; // 160 seconds
        // Set up the next scan interval in an exponential backoff fashion.
        mPeriodicSingleScanInterval *= 2;
        if (mPeriodicSingleScanInterval >  MAX_PERIODIC_SCAN_INTERVAL_MS) {
            mPeriodicSingleScanInterval = MAX_PERIODIC_SCAN_INTERVAL_MS;
        }
    }
	    
		
3. pno扫描
	   private void startDisconnectedPnoScan() {
        // TODO(b/29503772): Need to change this interface.

        // Initialize PNO settings
        PnoSettings pnoSettings = new PnoSettings();
        List<PnoSettings.PnoNetwork> pnoNetworkList = mConfigManager.retrievePnoNetworkList();
        int listSize = pnoNetworkList.size();

        if (listSize == 0) {
            // No saved network
            localLog("No saved network for starting disconnected PNO.");
            return;
        }

        pnoSettings.networkList = new PnoSettings.PnoNetwork[listSize];
        pnoSettings.networkList = pnoNetworkList.toArray(pnoSettings.networkList);
        pnoSettings.min5GHzRssi = mMin5GHzRssi;
        pnoSettings.min24GHzRssi = mMin24GHzRssi;
        pnoSettings.initialScoreMax = mInitialScoreMax;
        pnoSettings.currentConnectionBonus = mCurrentConnectionBonus;
        pnoSettings.sameNetworkBonus = mSameNetworkBonus;
        pnoSettings.secureBonus = mSecureBonus;
        pnoSettings.band5GHzBonus = mBand5GHzBonus;

        // Initialize scan settings
        ScanSettings scanSettings = new ScanSettings();
        scanSettings.band = getScanBand();
        scanSettings.reportEvents = WifiScanner.REPORT_EVENT_NO_BATCH;
        scanSettings.numBssidsPerScan = 0;
        scanSettings.periodInMs = DISCONNECTED_PNO_SCAN_INTERVAL_MS;			// 20s

        mPnoScanListener.clearScanDetails();

        mScanner.startDisconnectedPnoScan(scanSettings, pnoSettings, mPnoScanListener);
        mPnoScanStarted = true;
    }
	待续。。。
```