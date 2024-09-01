```
Android设备在于AP关联时，如果AP返回关联拒绝帧，Android设别会把AP加入黑名单中。
黑名单中的设备将会在扫描时，延时一段时间放在后面处理。
代码以及log基于SDM450, Android O
10-26 22:49:44.937  2215  2215 D wpa_supplicant: nl80211: Drv Event 46 (NL80211_CMD_CONNECT) received for wlan0
10-26 22:49:44.937  2215  2215 D wpa_supplicant: nl80211: Connect event (status=1 ignore_next_local_disconnect=0)
10-26 22:49:44.937  2215  2215 D wpa_supplicant: wlan0: Event ASSOC_REJECT (13) received
10-26 22:49:44.937  2215  2215 I wpa_supplicant: wlan0: CTRL-EVENT-ASSOC-REJECT bssid=ac:a3:1e:90:a7:00 status_code=1
10-26 22:49:44.937  2215  2215 D wpa_supplicant: CTRL-DEBUG: ctrl_sock-sendmsg: sock=8 sndbuf=163840 outq=0 send_len=61
10-26 22:49:44.937  2215  2215 D wpa_supplicant: CTRL_IFACE monitor sent successfully to /data/misc/wifi/sockets/wpa_ctrl_1749-3\x00
10-26 22:49:44.937  2215  2215 D wpa_supplicant: wlan0: Radio work 'connect'@0xab316240 done in 1.284062 seconds
10-26 22:49:44.937  2215  2215 D wpa_supplicant: wlan0: radio_work_free('connect'@0xab316240: num_active_works --> 0
10-26 22:49:44.937  1749  2340 D WifiMonitor: Event [IFNAME=wlan0 CTRL-EVENT-ASSOC-REJECT bssid=ac:a3:1e:90:a7:00 status_code=1]
10-26 22:49:44.937  2215  2215 D wpa_supplicant: Added BSSID ac:a3:1e:90:a7:00 into blacklist
10-26 22:49:44.938  2215  2215 D wpa_supplicant: wlan0: Another BSS in this ESS has been seen; try it next		// ESS中还有其他的BSS
10-26 22:49:44.938  2215  2215 D wpa_supplicant: BSSID ac:a3:1e:90:a7:00 blacklist count incremented to 2
10-26 22:49:44.938  2215  2215 D wpa_supplicant: wlan0: Blacklist count 3 --> request scan in 1000 ms
10-26 22:49:44.938  2215  2215 D wpa_supplicant: wlan0: Setting scan request: 1.000000 sec
10-26 22:49:44.938  2215  2215 D wpa_supplicant: wlan0: State: ASSOCIATING -> DISCONNECTED

10-26 22:49:43.094  1749  2340 D WifiMonitor: Event [IFNAME=wlan0 CTRL-EVENT-ASSOC-REJECT bssid=ac:a3:1e:90:a7:00 status_code=1]
10-26 22:49:43.094  1749  2340 D WifiMonitor: wlan0 cnt=5727 dispatchEvent: CTRL-EVENT-ASSOC-REJECT bssid=ac:a3:1e:90:a7:00 status_code=1
10-26 22:49:43.095  1749  2340 D WifiMonitor: Event [IFNAME=wlan0 CTRL-EVENT-STATE-CHANGE id=0 state=0 BSSID=00:00:00:00:00:00 SSID=coupang]
10-26 22:49:43.095  1749  2340 D WifiMonitor: wlan0 cnt=5728 dispatchEvent: CTRL-EVENT-STATE-CHANGE id=0 state=0 BSSID=00:00:00:00:00:00 SSID=coupang
10-26 22:49:43.095  1749  2081 D WifiStateMachine:  DisconnectedState ASSOCIATION_REJECTION_EVENT  rt=13253916/14362444 5727 1 ac:a3:1e:90:a7:00 blacklist=false
10-26 22:49:43.095  1749  2081 D WifiStateMachine:  ConnectModeState ASSOCIATION_REJECTION_EVENT  rt=13253916/14362445 5727 1 ac:a3:1e:90:a7:00 blacklist=false

external\wpa_supplicant_8\wpa_supplicant\events.c
void wpa_supplicant_event(void *ctx, enum wpa_event_type event,
			  union wpa_event_data *data)
{

	struct wpa_supplicant *wpa_s = ctx;
	int resched;

	if (wpa_s->wpa_state == WPA_INTERFACE_DISABLED &&
	    event != EVENT_INTERFACE_ENABLED &&
	    event != EVENT_INTERFACE_STATUS &&
	    event != EVENT_SCAN_RESULTS &&
	    event != EVENT_SCHED_SCAN_STOPPED) {
		wpa_dbg(wpa_s, MSG_DEBUG,
			"Ignore event %s (%d) while interface is disabled",
			event_to_string(event), event);
		return;
	}

#ifndef CONFIG_NO_STDOUT_DEBUG
{
	int level = MSG_DEBUG;

	if (event == EVENT_RX_MGMT && data->rx_mgmt.frame_len >= 24) {
		const struct ieee80211_hdr *hdr;
		u16 fc;
		hdr = (const struct ieee80211_hdr *) data->rx_mgmt.frame;
		fc = le_to_host16(hdr->frame_control);
		if (WLAN_FC_GET_TYPE(fc) == WLAN_FC_TYPE_MGMT &&
		    WLAN_FC_GET_STYPE(fc) == WLAN_FC_STYPE_BEACON)
			level = MSG_EXCESSIVE;
	}
	//打印			Event ASSOC_REJECT (13) received
	wpa_dbg(wpa_s, level, "Event %s (%d) received",
		event_to_string(event), event);
}

	case EVENT_ASSOC_REJECT:                // 处理关联拒绝事件
		if (data->assoc_reject.bssid)
		//打印结果：wlan0: CTRL-EVENT-ASSOC-REJECT bssid=ac:a3:1e:90:a7:00 status_code=1
			wpa_msg(wpa_s, MSG_INFO, WPA_EVENT_ASSOC_REJECT
				"bssid=" MACSTR	" status_code=%u%s%s%s",
				MAC2STR(data->assoc_reject.bssid),
				data->assoc_reject.status_code,
				data->assoc_reject.timed_out ? " timeout" : "",
				data->assoc_reject.timeout_reason ? "=" : "",
				data->assoc_reject.timeout_reason ?
				data->assoc_reject.timeout_reason : "");
		else
			wpa_msg(wpa_s, MSG_INFO, WPA_EVENT_ASSOC_REJECT
				"status_code=%u%s%s%s",
				data->assoc_reject.status_code,
				data->assoc_reject.timed_out ? " timeout" : "",
				data->assoc_reject.timeout_reason ? "=" : "",
				data->assoc_reject.timeout_reason ?
				data->assoc_reject.timeout_reason : "");
		wpa_s->assoc_status_code = data->assoc_reject.status_code;
		wpa_s->assoc_timed_out = data->assoc_reject.timed_out;
		wpas_notify_assoc_status_code(wpa_s);
		if (wpa_s->drv_flags & WPA_DRIVER_FLAGS_SME)
			sme_event_assoc_reject(wpa_s, data);
		else {
			const u8 *bssid = data->assoc_reject.bssid;

#ifdef CONFIG_FILS
			/* Update ERP next sequence number */
			if (wpa_s->auth_alg == WPA_AUTH_ALG_FILS)
				eapol_sm_update_erp_next_seq_num(
				      wpa_s->eapol,
				      data->assoc_reject.fils_erp_next_seq_num);
#endif /* CONFIG_FILS */

			if (bssid == NULL || is_zero_ether_addr(bssid))
				bssid = wpa_s->pending_bssid;
			wpas_connection_failed(wpa_s, bssid);
			wpa_supplicant_mark_disassoc(wpa_s);
		}
		break;

			wpas_connection_failed(wpa_s, bssid);
			wpa_supplicant_mark_disassoc(wpa_s);
}

external\wpa_supplicant_8\wpa_supplicant\wpa_supplicant.c
void wpas_connection_failed(struct wpa_supplicant *wpa_s, const u8 *bssid)
{
	int timeout;
	int count;
	int *freqs = NULL;

	wpas_connect_work_done(wpa_s);

	/*
	 * Remove possible authentication timeout since the connection failed.
	 */
	eloop_cancel_timeout(wpa_supplicant_timeout, wpa_s, NULL);

	/*
	 * There is no point in blacklisting the AP if this event is
	 * generated based on local request to disconnect.
	 */
	if (wpa_s->own_disconnect_req) {
		wpa_s->own_disconnect_req = 0;
		wpa_dbg(wpa_s, MSG_DEBUG,
			"Ignore connection failure due to local request to disconnect");
		return;
	}
	if (wpa_s->disconnected) {
		wpa_dbg(wpa_s, MSG_DEBUG, "Ignore connection failure "
			"indication since interface has been put into "
			"disconnected state");
		return;
	}

	/*
	 * Add the failed BSSID into the blacklist and speed up next scan
	 * attempt if there could be other APs that could accept association.
	 * The current blacklist count indicates how many times we have tried
	 * connecting to this AP and multiple attempts mean that other APs are
	 * either not available or has already been tried, so that we can start
	 * increasing the delay here to avoid constant scanning.
	 */
	count = wpa_blacklist_add(wpa_s, bssid);		// 加入黑名单
	if (count == 1 && wpa_s->current_bss) {
		/*
		 * This BSS was not in the blacklist before. If there is
		 * another BSS available for the same ESS, we should try that
		 * next. Otherwise, we may as well try this one once more
		 * before allowing other, likely worse, ESSes to be considered.
		 */
		 // 一个ESS中有多个BSS，ssid都相同，由不同的AP组成，BSSID不同。
		 // 这个函数就是从BSS中找出其他频率,没有加入黑名单的BSSID来进行下一次扫描。
		freqs = get_bss_freqs_in_ess(wpa_s);
		if (freqs) {
			wpa_dbg(wpa_s, MSG_DEBUG, "Another BSS in this ESS "
				"has been seen; try it next");
			wpa_blacklist_add(wpa_s, bssid);
			/*
			 * On the next scan, go through only the known channels
			 * used in this ESS based on previous scans to speed up
			 * common load balancing use case.
			 */
			os_free(wpa_s->next_scan_freqs);
			// 下次扫描的频率
			wpa_s->next_scan_freqs = freqs;
		}
	}

	/*
	 * Add previous failure count in case the temporary blacklist was
	 * cleared due to no other BSSes being available.
	 */
	count += wpa_s->extra_blacklist_count;

	if (count > 3 && wpa_s->current_ssid) {
		wpa_printf(MSG_DEBUG, "Continuous association failures - "
			   "consider temporary network disabling");
		wpas_auth_failed(wpa_s, "CONN_FAILED");
	}

	switch (count) {
	case 1:
		timeout = 100;
		break;
	case 2:
		timeout = 500;
		break;
	case 3:
		timeout = 1000;
		break;
	case 4:
		timeout = 5000;
		break;
	default:
		timeout = 10000;
		break;
	}

	wpa_dbg(wpa_s, MSG_DEBUG, "Blacklist count %d --> request scan in %d "
		"ms", count, timeout);

	/*
	 * TODO: if more than one possible AP is available in scan results,
	 * could try the other ones before requesting a new scan.
	 */
	wpa_supplicant_req_scan(wpa_s, timeout / 1000,
				1000 * (timeout % 1000));
}

void wpa_supplicant_mark_disassoc(struct wpa_supplicant *wpa_s)
{
	int bssid_changed;

	wnm_bss_keep_alive_deinit(wpa_s);

#ifdef CONFIG_IBSS_RSN
	ibss_rsn_deinit(wpa_s->ibss_rsn);
	wpa_s->ibss_rsn = NULL;
#endif /* CONFIG_IBSS_RSN */

#ifdef CONFIG_AP
	wpa_supplicant_ap_deinit(wpa_s);
#endif /* CONFIG_AP */

#ifdef CONFIG_HS20
	/* Clear possibly configured frame filters */
	wpa_drv_configure_frame_filters(wpa_s, 0);
#endif /* CONFIG_HS20 */

	if (wpa_s->wpa_state == WPA_INTERFACE_DISABLED)
		return;
	// 更改wpa的状态。
	wpa_supplicant_set_state(wpa_s, WPA_DISCONNECTED);
	bssid_changed = !is_zero_ether_addr(wpa_s->bssid);
	os_memset(wpa_s->bssid, 0, ETH_ALEN);
	os_memset(wpa_s->pending_bssid, 0, ETH_ALEN);
	sme_clear_on_disassoc(wpa_s);
	wpa_s->current_bss = NULL;
	wpa_s->assoc_freq = 0;

	if (bssid_changed)
		wpas_notify_bssid_changed(wpa_s);

	eapol_sm_notify_portEnabled(wpa_s->eapol, FALSE);
	eapol_sm_notify_portValid(wpa_s->eapol, FALSE);
	if (wpa_key_mgmt_wpa_psk(wpa_s->key_mgmt) ||
	    wpa_s->key_mgmt == WPA_KEY_MGMT_OWE ||
	    wpa_s->key_mgmt == WPA_KEY_MGMT_DPP)
		eapol_sm_notify_eap_success(wpa_s->eapol, FALSE);
	wpa_s->ap_ies_from_associnfo = 0;
	wpa_s->current_ssid = NULL;
	eapol_sm_notify_config(wpa_s->eapol, NULL, NULL);
	wpa_s->key_mgmt = 0;

	wpas_rrm_reset(wpa_s);
	wpa_s->wnmsleep_used = 0;
}

external\wpa_supplicant_8\wpa_supplicant\wpa_supplicant.c
void wpa_supplicant_set_state(struct wpa_supplicant *wpa_s,
			      enum wpa_states state)
{
	enum wpa_states old_state = wpa_s->wpa_state;
	                                        // 这里就是我们经常看到的log： wlan0: State: ASSOCIATING -> DISCONNECTED
	wpa_dbg(wpa_s, MSG_DEBUG, "State: %s -> %s",
		wpa_supplicant_state_txt(wpa_s->wpa_state),
		wpa_supplicant_state_txt(state));

	if (state == WPA_INTERFACE_DISABLED) {
		/* Assure normal scan when interface is restored */
		wpa_s->normal_scans = 0;
	}

	if (state == WPA_COMPLETED) {
		wpas_connect_work_done(wpa_s);
		/* Reinitialize normal_scan counter */
		wpa_s->normal_scans = 0;
	}

......

	if (state != WPA_SCANNING)
		wpa_supplicant_notify_scanning(wpa_s, 0);
	                                                                            // 如果是WPA_COMPLETED
	if (state == WPA_COMPLETED && wpa_s->new_connection) {
		struct wpa_ssid *ssid = wpa_s->current_ssid;
		int fils_hlp_sent = 0;

#ifdef CONFIG_SME
		if ((wpa_s->drv_flags & WPA_DRIVER_FLAGS_SME) &&
		    wpa_auth_alg_fils(wpa_s->sme.auth_alg))
			fils_hlp_sent = 1;
#endif /* CONFIG_SME */
		if (!(wpa_s->drv_flags & WPA_DRIVER_FLAGS_SME) &&
		    wpa_auth_alg_fils(wpa_s->auth_alg))
			fils_hlp_sent = 1;

#if defined(CONFIG_CTRL_IFACE) || !defined(CONFIG_NO_STDOUT_DEBUG)
		wpa_msg(wpa_s, MSG_INFO, WPA_EVENT_CONNECTED "- Connection to "
			MACSTR " completed [id=%d id_str=%s%s]",
			MAC2STR(wpa_s->bssid),
			ssid ? ssid->id : -1,
			ssid && ssid->id_str ? ssid->id_str : "",
			fils_hlp_sent ? " FILS_HLP_SENT" : "");
#endif /* CONFIG_CTRL_IFACE || !CONFIG_NO_STDOUT_DEBUG */
		wpas_clear_temp_disabled(wpa_s, ssid, 1);
		wpa_blacklist_clear(wpa_s);			// 连接上了，设备从黑名单移除
		wpa_s->extra_blacklist_count = 0;
		wpa_s->new_connection = 0;
		wpa_drv_set_operstate(wpa_s, 1);
#ifndef IEEE8021X_EAPOL
		wpa_drv_set_supp_port(wpa_s, 1);
#endif /* IEEE8021X_EAPOL */
		wpa_s->after_wps = 0;
		wpa_s->known_wps_freq = 0;
		wpas_p2p_completed(wpa_s);

		sme_sched_obss_scan(wpa_s, 1);

#if defined(CONFIG_FILS) && defined(IEEE8021X_EAPOL)
		if (!fils_hlp_sent && ssid && ssid->eap.erp)
			wpas_update_fils_connect_params(wpa_s);
#endif /* CONFIG_FILS && IEEE8021X_EAPOL */
	// disconnect 状态的处理
	} else if (state == WPA_DISCONNECTED || state == WPA_ASSOCIATING ||
		   state == WPA_ASSOCIATED) {
		wpa_s->new_connection = 1;
		wpa_drv_set_operstate(wpa_s, 0);
#ifndef IEEE8021X_EAPOL
		wpa_drv_set_supp_port(wpa_s, 0);
#endif /* IEEE8021X_EAPOL */
		sme_sched_obss_scan(wpa_s, 0);
	}
	wpa_s->wpa_state = state;

#ifdef CONFIG_BGSCAN
	if (state == WPA_COMPLETED)
		wpa_supplicant_start_bgscan(wpa_s);
	else if (state < WPA_ASSOCIATED)
		wpa_supplicant_stop_bgscan(wpa_s);
#endif /* CONFIG_BGSCAN */
#if 0 //qc_modify
	wpa_s->wapi_state = wpa_state_to_wapi_state(wpa_s->wpa_state);
	wpa_printf(MSG_DEBUG, "%s wapi_state : %d\n",__func__ ,wpa_s->wapi_state);
#endif
	if (state == WPA_AUTHENTICATING)
		wpa_supplicant_stop_autoscan(wpa_s);
	// 断开连接，进行自动扫描
	if (state == WPA_DISCONNECTED || state == WPA_INACTIVE)
		wpa_supplicant_start_autoscan(wpa_s);

	if (old_state >= WPA_ASSOCIATED && wpa_s->wpa_state < WPA_ASSOCIATED)
		wmm_ac_notify_disassoc(wpa_s);

	if (wpa_s->wpa_state != old_state) {
		wpas_notify_state_changed(wpa_s, wpa_s->wpa_state, old_state);

		/*
		 * Notify the P2P Device interface about a state change in one
		 * of the interfaces.
		 */
		wpas_p2p_indicate_state_change(wpa_s);

		if (wpa_s->wpa_state == WPA_COMPLETED ||
		    old_state == WPA_COMPLETED)
			wpas_notify_auth_changed(wpa_s);
	}
}

// 通知时间给上层
external\wpa_supplicant_8\wpa_supplicant\notify.c
void wpas_notify_state_changed(struct wpa_supplicant *wpa_s,
			       enum wpa_states new_state,
			       enum wpa_states old_state)
{
	if (wpa_s->p2p_mgmt)
		return;

	/* notify the old DBus API */
	wpa_supplicant_dbus_notify_state_change(wpa_s, new_state,
						old_state);

	/* notify the new DBus API */
	wpas_dbus_signal_prop_changed(wpa_s, WPAS_DBUS_PROP_STATE);

#ifdef CONFIG_FST
	if (wpa_s->fst && !is_zero_ether_addr(wpa_s->bssid)) {
		if (new_state == WPA_COMPLETED)
			fst_notify_peer_connected(wpa_s->fst, wpa_s->bssid);
		else if (old_state >= WPA_ASSOCIATED &&
			 new_state < WPA_ASSOCIATED)
			fst_notify_peer_disconnected(wpa_s->fst, wpa_s->bssid);
	}
#endif /* CONFIG_FST */

	if (new_state == WPA_COMPLETED)
		wpas_p2p_notif_connected(wpa_s);
	else if (old_state >= WPA_ASSOCIATED && new_state < WPA_ASSOCIATED)
		wpas_p2p_notif_disconnected(wpa_s);

	sme_state_changed(wpa_s);

#ifdef ANDROID
	wpa_msg_ctrl(wpa_s, MSG_INFO, WPA_EVENT_STATE_CHANGE
		     "id=%d state=%d BSSID=" MACSTR " SSID=%s",
		     wpa_s->current_ssid ? wpa_s->current_ssid->id : -1,
		     new_state,
		     MAC2STR(wpa_s->bssid),
		     wpa_s->current_ssid && wpa_s->current_ssid->ssid ?
		     wpa_ssid_txt(wpa_s->current_ssid->ssid,
				  wpa_s->current_ssid->ssid_len) : "");
#endif /* ANDROID */

	wpas_hidl_notify_state_changed(wpa_s);                // 发送给hidl
}

external\wpa_supplicant_8\wpa_supplicant\hidl\1.0\hidl.cpp
int wpas_hidl_notify_state_changed(struct wpa_supplicant *wpa_s)
{
	if (!wpa_s || !wpa_s->global->hidl)
		return 1;

	wpa_printf(
	    MSG_DEBUG, "Notifying state change event to hidl control: %s",
	    wpa_supplicant_state_txt(wpa_s->wpa_state));

	HidlManager *hidl_manager = HidlManager::getInstance();
	if (!hidl_manager)
		return 1;

	return hidl_manager->notifyStateChange(wpa_s);
}

external\wpa_supplicant_8\wpa_supplicant\hidl\1.0\hidl_manager.cpp
int HidlManager::notifyStateChange(struct wpa_supplicant *wpa_s)
{
	if (!wpa_s)
		return 1;

	if (sta_iface_object_map_.find(wpa_s->ifname) ==
	    sta_iface_object_map_.end())
		return 1;

	// Invoke the |onStateChanged| method on all registered callbacks.
	uint32_t hidl_network_id = UINT32_MAX;
	std::vector<uint8_t> hidl_ssid;
	if (wpa_s->current_ssid) {
		hidl_network_id = wpa_s->current_ssid->id;
		hidl_ssid.assign(
		    wpa_s->current_ssid->ssid,
		    wpa_s->current_ssid->ssid + wpa_s->current_ssid->ssid_len);
	}
	uint8_t *bssid;
	// wpa_supplicant sets the |pending_bssid| field when it starts a
	// connection. Only after association state does it update the |bssid|
	// field. So, in the HIDL callback send the appropriate bssid.
	if (wpa_s->wpa_state <= WPA_ASSOCIATED) {
		bssid = wpa_s->pending_bssid;
	} else {
		bssid = wpa_s->bssid;
	}
	bool fils_hlp_sent =
		(wpa_auth_alg_fils(wpa_s->auth_alg) &&
		 !dl_list_empty(&wpa_s->fils_hlp_req) &&
		 (wpa_s->wpa_state == WPA_COMPLETED)) ? true : false;

    // 回调函数
	if (checkForVendorStaIfaceCallback(wpa_s->ifname) == true) {
		callWithEachVendorStaIfaceCallback(
		    wpa_s->ifname, std::bind(
		       &ISupplicantVendorStaIfaceCallback::onVendorStateChanged,
		       std::placeholders::_1,
		       static_cast<ISupplicantStaIfaceCallback::State>(
		       wpa_s->wpa_state),
		       bssid, hidl_network_id, hidl_ssid, fils_hlp_sent));
	} else {
		callWithEachStaIfaceCallback(
		    wpa_s->ifname, std::bind(
			&ISupplicantStaIfaceCallback::onStateChanged,
			std::placeholders::_1,
			static_cast<ISupplicantStaIfaceCallback::State>(
			wpa_s->wpa_state),
			bssid, hidl_network_id, hidl_ssid));
	}

	return 0;
}

frameworks\opt\net\wifi\service\java\com\android\server\wifi\SupplicantStaIfaceHal.java
   private class SupplicantVendorStaIfaceHalCallback extends ISupplicantVendorStaIfaceCallback.Stub {
   
          @Override
        public void onAssociationRejected(byte[/* 6 */] bssid, int statusCode, boolean timedOut) {
            synchronized (mLock) {
                logCallback("onAssociationRejected");
                mWifiMonitor.broadcastAssociationRejectionEvent(mIfaceName, statusCode, timedOut,
                        NativeUtil.macAddressFromByteArray(bssid));
            }
        }
		
	// 发送消息给WifiStateMachine
frameworks\opt\net\wifi\service\java\com\android\server\wifi\WifiMonitor.java
    public void broadcastAssociationRejectionEvent(String iface, int status, boolean timedOut,
                                                   String bssid) {
        sendMessage(iface, ASSOCIATION_REJECTION_EVENT, timedOut ? 1 : 0, status, bssid);
    }
	
	
frameworks\opt\net\wifi\service\java\com\android\server\wifi\WifiStateMachine.java	
	        public boolean processMessage(Message message) {
            WifiConfiguration config;
            int netId;
            boolean ok;
            boolean didDisconnect;
            String bssid;
            String ssid;
            NetworkUpdateResult result;
            Set<Integer> removedNetworkIds;
            int reasonCode;
            boolean timedOut;
            logStateAndMessage(message, this);

            switch (message.what) {
                case WifiMonitor.ASSOCIATION_REJECTION_EVENT:
                    mWifiDiagnostics.captureBugReportData(
                            WifiDiagnostics.REPORT_REASON_ASSOC_FAILURE);
                    didBlackListBSSID = false;
                    bssid = (String) message.obj;
                    timedOut = message.arg1 > 0;
                    reasonCode = message.arg2;
                    Log.d(TAG, "Assocation Rejection event: bssid=" + bssid + " reason code="
                            + reasonCode + " timedOut=" + Boolean.toString(timedOut));
                    if (bssid == null || TextUtils.isEmpty(bssid)) {
                        // If BSSID is null, use the target roam BSSID
                        bssid = mTargetRoamBSSID;
                    }
                    if (bssid != null) {
                        // If we have a BSSID, tell configStore to black list it
                        didBlackListBSSID = mWifiConnectivityManager.trackBssid(bssid, false,
                            reasonCode);
                    }
                    // 根据reason code更新状态
                    mWifiConfigManager.updateNetworkSelectionStatus(mTargetNetworkId,
                            WifiConfiguration.NetworkSelectionStatus
                            .DISABLED_ASSOCIATION_REJECTION);
                    mWifiConfigManager.setRecentFailureAssociationStatus(mTargetNetworkId,
                            reasonCode);
                    mSupplicantStateTracker.sendMessage(WifiMonitor.ASSOCIATION_REJECTION_EVENT);
                    // If rejection occurred while Metrics is tracking a ConnnectionEvent, end it.
                    reportConnectionAttemptEnd(
                            WifiMetrics.ConnectionEvent.FAILURE_ASSOCIATION_REJECTION,
                            WifiMetricsProto.ConnectionEvent.HLF_NONE);
                    mWifiInjector.getWifiLastResortWatchdog()
                            .noteConnectionFailureAndTriggerIfNeeded(
                                    getTargetSsid(), bssid,
                                    WifiLastResortWatchdog.FAILURE_CODE_ASSOCIATION);
                    break;
					
							
public boolean trackBssid(String bssid, boolean enable, int reasonCode) {
        localLog("trackBssid: " + (enable ? "enable " : "disable ") + bssid + " reason code "
                + reasonCode);

        if (bssid == null) {
            return false;
        }
                    // 加入黑名单
        if (!updateBssidBlacklist(bssid, enable, reasonCode)) {
            return false;
        }

        // Blacklist was updated, so update firmware roaming configuration.
        updateFirmwareRoamingConfiguration();
                // 立即扫描
        if (!enable) {
            // Disabling a BSSID can happen when connection to the AP was rejected.
            // We start another scan immediately so that WifiNetworkSelector can
            // give us another candidate to connect to.
            startConnectivityScan(SCAN_IMMEDIATELY);
        }

        return true;
    }
```