从android N开始，引入了wifi评分机制，选择wifi的时候会通过评分来选择。

android O源码

frameworks\opt\net\wifi\service\java\com\android\server\wifi\SavedNetworkEvaluator.java
```
   private int calculateBssidScore(ScanResult scanResult, WifiConfiguration network,
                        WifiConfiguration currentNetwork, String currentBssid,
                        StringBuffer sbuf) {
        int score = 0;
        // 判断信号是否为5GHz
        boolean is5GHz = scanResult.is5GHz();

        sbuf.append("[ ").append(scanResult.SSID).append(" ").append(scanResult.BSSID)
                .append(" RSSI:").append(scanResult.level).append(" ] ");
        // Calculate the RSSI score.
        int rssiSaturationThreshold = is5GHz ? mThresholdSaturatedRssi5 : mThresholdSaturatedRssi24;
        // 根据wifi的信号等级来获取rssi
        int rssi = scanResult.level < rssiSaturationThreshold ? scanResult.level
                : rssiSaturationThreshold;
        score += (rssi + mRssiScoreOffset) * mRssiScoreSlope;
        sbuf.append(" RSSI score: ").append(score).append(",");

        // 5GHz band bonus.
        if (is5GHz) {
            score += mBand5GHzAward; // score += 40
            sbuf.append(" 5GHz bonus: ").append(mBand5GHzAward).append(",");
        }
        // 之前连接过会有奖励
        // Last user selection award.
        int lastUserSelectedNetworkId = mWifiConfigManager.getLastSelectedNetwork();
        if (lastUserSelectedNetworkId != WifiConfiguration.INVALID_NETWORK_ID
                && lastUserSelectedNetworkId == network.networkId) {
            // 开机时间减去连接的时间戳,得到的时间就是连接的时间到现在总共多久
            long timeDifference = mClock.getElapsedSinceBootMillis()
                    - mWifiConfigManager.getLastSelectedTimeStamp();
            if (timeDifference > 0) {
                int bonus = mLastSelectionAward - (int) (timeDifference / 1000 / 60);
                score += bonus > 0 ? bonus : 0;
                sbuf.append(" User selection ").append(timeDifference / 1000 / 60)
                        .append(" minutes ago, bonus: ").append(bonus).append(",");
            }
        }

        // Same network award.
        // 当前正在连接的SSID也会有奖励
        if (currentNetwork != null
                && (network.networkId == currentNetwork.networkId
                //TODO(b/36788683): re-enable linked configuration check
                /* || network.isLinked(currentNetwork) */)) {
            score += mSameNetworkAward;
            sbuf.append(" Same network bonus: ").append(mSameNetworkAward).append(",");

            // 如果支持漫游，也会有漫游奖励
            // When firmware roaming is supported, equivalent BSSIDs (the ones under the
            // same network as the currently connected one) get the same BSSID award.
            // 漫游奖励，SSID相同，BSSID不同。
            if (mConnectivityHelper.isFirmwareRoamingSupported()
                    && currentBssid != null && !currentBssid.equals(scanResult.BSSID)) {
                score += mSameBssidAward;
                sbuf.append(" Equivalent BSSID bonus: ").append(mSameBssidAward).append(",");
            }
        }

        // Same BSSID award.
        if (currentBssid != null && currentBssid.equals(scanResult.BSSID)) {
            score += mSameBssidAward;
            sbuf.append(" Same BSSID bonus: ").append(mSameBssidAward).append(",");
        }

        // Security award.
        // 不是开放的网络，也会有奖励
        if (!WifiConfigurationUtil.isConfigForOpenNetwork(network)) {
            score += mSecurityAward;
            sbuf.append(" Secure network bonus: ").append(mSecurityAward).append(",");
        }

        sbuf.append(" ## Total score: ").append(score).append("\n");

        return score;
    }

    /**
     * Evaluate all the networks from the scan results and return
     * the WifiConfiguration of the network chosen for connection.
     *
     * @return configuration of the chosen network;
     *         null if no network in this category is available.
     */
    public WifiConfiguration evaluateNetworks(List<ScanDetail> scanDetails,
                    WifiConfiguration currentNetwork, String currentBssid, boolean connected,
                    boolean untrustedNetworkAllowed,
                    List<Pair<ScanDetail, WifiConfiguration>> connectableNetworks) {
        int highestScore = Integer.MIN_VALUE;
        ScanResult scanResultCandidate = null;
        WifiConfiguration candidate = null;
        StringBuffer scoreHistory = new StringBuffer();

        for (ScanDetail scanDetail : scanDetails) {
            ScanResult scanResult = scanDetail.getScanResult();
            int highestScoreOfScanResult = Integer.MIN_VALUE;       // 0x80000000
            int candidateIdOfScanResult = WifiConfiguration.INVALID_NETWORK_ID;  // -1

            // One ScanResult can be associated with more than one networks, hence we calculate all
            // the scores and use the highest one as the ScanResult's score.
            List<WifiConfiguration> associatedConfigurations = null;
            WifiConfiguration associatedConfiguration =
                    mWifiConfigManager.getConfiguredNetworkForScanDetailAndCache(scanDetail);

            if (associatedConfiguration == null) {
                continue;
            } else {
                associatedConfigurations =
                    new ArrayList<>(Arrays.asList(associatedConfiguration));
            }

            for (WifiConfiguration network : associatedConfigurations) {
                /**
                 * Ignore Passpoint and Ephemeral networks. They are configured networks,
                 * but without being persisted to the storage. They are evaluated by
                 * {@link PasspointNetworkEvaluator} and {@link ScoredNetworkEvaluator}
                 * respectively.
                 */
                if (network.isPasspoint() || network.isEphemeral()) {
                    continue;
                }

                WifiConfiguration.NetworkSelectionStatus status =
                        network.getNetworkSelectionStatus();
                status.setSeenInLastQualifiedNetworkSelection(true);

                if (!status.isNetworkEnabled()) {
                    continue;
                } else if (network.BSSID != null &&  !network.BSSID.equals("any")
                        && !network.BSSID.equals(scanResult.BSSID)) {
                    // App has specified the only BSSID to connect for this
                    // configuration. So only the matching ScanResult can be a candidate.
                    localLog("Network " + WifiNetworkSelector.toNetworkString(network)
                            + " has specified BSSID " + network.BSSID + ". Skip "
                            + scanResult.BSSID);
                    continue;
                } else if (TelephonyUtil.isSimConfig(network)
                        && !mWifiConfigManager.isSimPresent()) {
                    // Don't select if security type is EAP SIM/AKA/AKA' when SIM is not present.
                    continue;
                }
                // 计算分数
                int score = calculateBssidScore(scanResult, network, currentNetwork, currentBssid,
                        scoreHistory);

                // Set candidate ScanResult for all saved networks to ensure that users can
                // override network selection. See WifiNetworkSelector#setUserConnectChoice.
                // TODO(b/36067705): consider alternative designs to push filtering/selecting of
                // user connect choice networks to RecommendedNetworkEvaluator.
                if (score > status.getCandidateScore() || (score == status.getCandidateScore()
                        && status.getCandidate() != null
                        && scanResult.level > status.getCandidate().level)) {
                    // 重新设置candidate
                    mWifiConfigManager.setNetworkCandidateScanResult(
                            network.networkId, scanResult, score);
                }

                // If the network is marked to use external scores, or is an open network with
                // curate saved open networks enabled, do not consider it for network selection.
                if (network.useExternalScores) {
                    localLog("Network " + WifiNetworkSelector.toNetworkString(network)
                            + " has external score.");
                    continue;
                }
                // 判断是否在扫描的列表中评分最高
                if (score > highestScoreOfScanResult) {
                    highestScoreOfScanResult = score;
                    // 保存networkId
                    candidateIdOfScanResult = network.networkId;
                }
            }

            if (connectableNetworks != null) {
                connectableNetworks.add(Pair.create(scanDetail,
                        mWifiConfigManager.getConfiguredNetwork(candidateIdOfScanResult)));
            }
            // 根据评分以及level来判断选择最合适的candidate
            if (highestScoreOfScanResult > highestScore
                    || (highestScoreOfScanResult == highestScore
                    && scanResultCandidate != null
                    && scanResult.level > scanResultCandidate.level)) {
                highestScore = highestScoreOfScanResult;
                scanResultCandidate = scanResult;
                mWifiConfigManager.setNetworkCandidateScanResult(
                        candidateIdOfScanResult, scanResultCandidate, highestScore);
                // Reload the network config with the updated info.
                candidate = mWifiConfigManager.getConfiguredNetwork(candidateIdOfScanResult);
            }
        }

        if (scoreHistory.length() > 0) {
            localLog("\n" + scoreHistory.toString());
        }

        if (scanResultCandidate == null) {
            localLog("did not see any good candidates.");
        }
        return candidate;
    }
}

```

2018-7-12, Shenzhen