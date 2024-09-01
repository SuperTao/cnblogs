```
扫描流程
1.如果之前就已经有相关记录，优化扫描，扫描记录部分的频率信道。
2.如果1中的扫描没有结果，清除黑名单中的进行选择。
3.如果2中没有结果，进行所有频率的信道进行扫描

相关log参考:
https://www.cnblogs.com/helloworldtoyou/p/9958084.html


external\wpa_supplicant_8\wpa_supplicant\src\drivers\driver_nl80211_event.c
static void do_process_drv_event(struct i802_bss *bss, int cmd,
				 struct nlattr **tb)
{
	struct wpa_driver_nl80211_data *drv = bss->drv;
	union wpa_event_data data;
	int external_scan_event = 0;

	wpa_printf(MSG_DEBUG, "nl80211: Drv Event %d (%s) received for %s",
		   cmd, nl80211_command_to_string(cmd), bss->ifname);

	if (cmd == NL80211_CMD_ROAM &&
	    (drv->capa.flags & WPA_DRIVER_FLAGS_KEY_MGMT_OFFLOAD)) {
		/*
		 * Device will use roam+auth vendor event to indicate
		 * roaming, so ignore the regular roam event.
		 */
		wpa_printf(MSG_DEBUG,
			   "nl80211: Ignore roam event (cmd=%d), device will use vendor event roam+auth",
			   cmd);
		return;
	}

	if (drv->ap_scan_as_station != NL80211_IFTYPE_UNSPECIFIED &&
	    (cmd == NL80211_CMD_NEW_SCAN_RESULTS ||
	     cmd == NL80211_CMD_SCAN_ABORTED)) {
		wpa_driver_nl80211_set_mode(drv->first_bss,
					    drv->ap_scan_as_station);
		drv->ap_scan_as_station = NL80211_IFTYPE_UNSPECIFIED;
	}

	switch (cmd) {
	case NL80211_CMD_TRIGGER_SCAN:
		wpa_dbg(drv->ctx, MSG_DEBUG, "nl80211: Scan trigger");
		drv->scan_state = SCAN_STARTED;
		if (drv->scan_for_auth) {
			/*
			 * Cannot indicate EVENT_SCAN_STARTED here since we skip
			 * EVENT_SCAN_RESULTS in scan_for_auth case and the
			 * upper layer implementation could get confused about
			 * scanning state.
			 */
			wpa_printf(MSG_DEBUG, "nl80211: Do not indicate scan-start event due to internal scan_for_auth");
			break;
		}
		wpa_supplicant_event(drv->ctx, EVENT_SCAN_STARTED, NULL);
		break;
	case NL80211_CMD_START_SCHED_SCAN:
		wpa_dbg(drv->ctx, MSG_DEBUG, "nl80211: Sched scan started");
		drv->scan_state = SCHED_SCAN_STARTED;
		break;
	case NL80211_CMD_SCHED_SCAN_STOPPED:
		wpa_dbg(drv->ctx, MSG_DEBUG, "nl80211: Sched scan stopped");
		drv->scan_state = SCHED_SCAN_STOPPED;
		wpa_supplicant_event(drv->ctx, EVENT_SCHED_SCAN_STOPPED, NULL);
		break;
	case NL80211_CMD_NEW_SCAN_RESULTS:
		wpa_dbg(drv->ctx, MSG_DEBUG,
			"nl80211: New scan results available");
		if (drv->last_scan_cmd != NL80211_CMD_VENDOR)
			drv->scan_state = SCAN_COMPLETED;
		drv->scan_complete_events = 1;
		if (drv->last_scan_cmd == NL80211_CMD_TRIGGER_SCAN) {
			eloop_cancel_timeout(wpa_driver_nl80211_scan_timeout,
					     drv, drv->ctx);
			drv->last_scan_cmd = 0;
		} else {
			external_scan_event = 1;
		}
		send_scan_event(drv, 0, tb, external_scan_event);
		break;
		
}

static void send_scan_event(struct wpa_driver_nl80211_data *drv, int aborted,
			    struct nlattr *tb[], int external_scan)
{
	union wpa_event_data event;
	struct nlattr *nl;
	int rem;
	struct scan_info *info;
#define MAX_REPORT_FREQS 50
	int freqs[MAX_REPORT_FREQS];
	int num_freqs = 0;

	if (!external_scan && drv->scan_for_auth) {
		drv->scan_for_auth = 0;
		wpa_printf(MSG_DEBUG, "nl80211: Scan results for missing "
			   "cfg80211 BSS entry");
		wpa_driver_nl80211_authenticate_retry(drv);
		return;
	}

	os_memset(&event, 0, sizeof(event));
	info = &event.scan_info;
	info->aborted = aborted;
	info->external_scan = external_scan;
	info->nl_scan_event = 1;

	if (tb[NL80211_ATTR_SCAN_SSIDS]) {
		nla_for_each_nested(nl, tb[NL80211_ATTR_SCAN_SSIDS], rem) {
			struct wpa_driver_scan_ssid *s =
				&info->ssids[info->num_ssids];
			s->ssid = nla_data(nl);
			s->ssid_len = nla_len(nl);
			wpa_printf(MSG_DEBUG, "nl80211: Scan probed for SSID '%s'",
				   wpa_ssid_txt(s->ssid, s->ssid_len));
			info->num_ssids++;
			if (info->num_ssids == WPAS_MAX_SCAN_SSIDS)
				break;
		}
	}
	if (tb[NL80211_ATTR_SCAN_FREQUENCIES]) {
		char msg[300], *pos, *end;
		int res;

		pos = msg;
		end = pos + sizeof(msg);
		*pos = '\0';

		nla_for_each_nested(nl, tb[NL80211_ATTR_SCAN_FREQUENCIES], rem)
		{
			freqs[num_freqs] = nla_get_u32(nl);
			res = os_snprintf(pos, end - pos, " %d",
					  freqs[num_freqs]);
			if (!os_snprintf_error(end - pos, res))
				pos += res;
			num_freqs++;
			if (num_freqs == MAX_REPORT_FREQS - 1)
				break;
		}
		info->freqs = freqs;
		info->num_freqs = num_freqs;
		// 显示扫面结果中包含的频率
		//10-26 20:35:38.999  2215  2215 D wpa_supplicant: wlan0: nl80211: New scan results available
        //10-26 20:35:39.000  2215  2215 D wpa_supplicant: nl80211: Scan included frequencies: 5805 5240 5745 5200 5180 5220 2437 5785 2462 57
		wpa_printf(MSG_DEBUG, "nl80211: Scan included frequencies:%s",
			   msg);
	}

	if (tb[NL80211_ATTR_SCAN_START_TIME_TSF] &&
	    tb[NL80211_ATTR_SCAN_START_TIME_TSF_BSSID]) {
		info->scan_start_tsf =
			nla_get_u64(tb[NL80211_ATTR_SCAN_START_TIME_TSF]);
		os_memcpy(info->scan_start_tsf_bssid,
			  nla_data(tb[NL80211_ATTR_SCAN_START_TIME_TSF_BSSID]),
			  ETH_ALEN);
	}

	wpa_supplicant_event(drv->ctx, EVENT_SCAN_RESULTS, &event);
}

Z:\eda51\external\wpa_supplicant_8\wpa_supplicant\events.c
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
    // wpa_supplicant: wlan0: Event SCAN_RESULTS (3) received
	wpa_dbg(wpa_s, level, "Event %s (%d) received",
		event_to_string(event), event);
}
#endif /* CONFIG_NO_STDOUT_DEBUG */

	switch (event) {
	case EVENT_AUTH:
#ifdef CONFIG_FST
		if (!wpas_fst_update_mbie(wpa_s, data->auth.ies,
					  data->auth.ies_len))
			wpa_printf(MSG_DEBUG,
				   "FST: MB IEs updated from auth IE");
#endif /* CONFIG_FST */
		sme_event_auth(wpa_s, data);
		break;
	case EVENT_ASSOC:
#ifdef CONFIG_TESTING_OPTIONS
		if (wpa_s->ignore_auth_resp) {
			wpa_printf(MSG_INFO,
				   "EVENT_ASSOC - ignore_auth_resp active!");
			break;
		}
#endif /* CONFIG_TESTING_OPTIONS */
		wpa_supplicant_event_assoc(wpa_s, data);
		if (data &&
		    (data->assoc_info.authorized ||
		     (!(wpa_s->drv_flags & WPA_DRIVER_FLAGS_SME) &&
		      wpa_fils_is_completed(wpa_s->wpa))))
			wpa_supplicant_event_assoc_auth(wpa_s, data);
		if (data) {
			wpa_msg(wpa_s, MSG_INFO,
				WPA_EVENT_SUBNET_STATUS_UPDATE "status=%u",
				data->assoc_info.subnet_status);
		}
		break;
		
			case EVENT_SCAN_RESULTS:
		if (wpa_s->wpa_state == WPA_INTERFACE_DISABLED) {
			wpa_s->scan_res_handler = NULL;
			wpa_s->own_scan_running = 0;
			wpa_s->radio->external_scan_running = 0;
			wpa_s->last_scan_req = NORMAL_SCAN_REQ;
			break;
		}

		if (!(data && data->scan_info.external_scan) &&
		    os_reltime_initialized(&wpa_s->scan_start_time)) {
			struct os_reltime now, diff;
			os_get_reltime(&now);
			os_reltime_sub(&now, &wpa_s->scan_start_time, &diff);
			wpa_s->scan_start_time.sec = 0;
			wpa_s->scan_start_time.usec = 0;
			wpa_dbg(wpa_s, MSG_DEBUG, "Scan completed in %ld.%06ld seconds",
				diff.sec, diff.usec);
		}
		if (wpa_supplicant_event_scan_results(wpa_s, data))
			break; /* interface may have been removed */
		if (!(data && data->scan_info.external_scan))
			wpa_s->own_scan_running = 0;
		if (data && data->scan_info.nl_scan_event)
			wpa_s->radio->external_scan_running = 0;
		radio_work_check_next(wpa_s);
		break;
		...
}

static int wpa_supplicant_event_scan_results(struct wpa_supplicant *wpa_s,
					     union wpa_event_data *data)
{
	struct wpa_supplicant *ifs;
	int res;

	res = _wpa_supplicant_event_scan_results(wpa_s, data, 1, 0);
	if (res == 2) {
		/*
		 * Interface may have been removed, so must not dereference
		 * wpa_s after this.
		 */
		return 1;
	}

	if (res < 0) {
		/*
		 * If no scan results could be fetched, then no need to
		 * notify those interfaces that did not actually request
		 * this scan. Similarly, if scan results started a new operation on this
		 * interface, do not notify other interfaces to avoid concurrent
		 * operations during a connection attempt.
		 */
		return 0;
	}

	/*
	 * Check other interfaces to see if they share the same radio. If
	 * so, they get updated with this same scan info.
	 */
	dl_list_for_each(ifs, &wpa_s->radio->ifaces, struct wpa_supplicant,
			 radio_list) {
		if (ifs != wpa_s) {
			wpa_printf(MSG_DEBUG, "%s: Updating scan results from "
				   "sibling", ifs->ifname);
			res = _wpa_supplicant_event_scan_results(ifs, data, 0,
								 res > 0);
			if (res < 0)
				return 0;
		}
	}

	return 0;
}

static int _wpa_supplicant_event_scan_results(struct wpa_supplicant *wpa_s,
					      union wpa_event_data *data,
					      int own_request, int update_only)
{
	struct wpa_scan_results *scan_res = NULL;
	int ret = 0;
	int ap = 0;
#ifndef CONFIG_NO_RANDOM_POOL
	size_t i, num;
#endif /* CONFIG_NO_RANDOM_POOL */

#ifdef CONFIG_AP
	if (wpa_s->ap_iface)
		ap = 1;
#endif /* CONFIG_AP */

	wpa_supplicant_notify_scanning(wpa_s, 0);

	scan_res = wpa_supplicant_get_scan_results(wpa_s,
						   data ? &data->scan_info :
						   NULL, 1);
	if (scan_res == NULL) {
		if (wpa_s->conf->ap_scan == 2 || ap ||
		    wpa_s->scan_res_handler == scan_only_handler)
			return -1;
		if (!own_request)
			return -1;
		if (data && data->scan_info.external_scan)
			return -1;
		// 扫描结果为空，从新扫描
		wpa_dbg(wpa_s, MSG_DEBUG, "Failed to get scan results - try "
			"scanning again");
		wpa_supplicant_req_new_scan(wpa_s, 1, 0);
		ret = -1;
		goto scan_work_done;
	}

#ifndef CONFIG_NO_RANDOM_POOL
	num = scan_res->num;
	if (num > 10)
		num = 10;
	for (i = 0; i < num; i++) {
#ifdef CONFIG_WAPI
		u8 buf[6];
#else
		u8 buf[5];
#endif
		struct wpa_scan_res *res = scan_res->res[i];
		buf[0] = res->bssid[5];
		buf[1] = res->qual & 0xff;
		buf[2] = res->noise & 0xff;
		buf[3] = res->level & 0xff;
		buf[4] = res->tsf & 0xff;
#ifdef CONFIG_WAPI
		buf[5] = res->wapi_ie_len & 0xff;
#endif
		random_add_randomness(buf, sizeof(buf));
	}
#endif /* CONFIG_NO_RANDOM_POOL */

	if (update_only) {
		ret = 1;
		goto scan_work_done;
	}

	if (own_request && wpa_s->scan_res_handler &&
	    !(data && data->scan_info.external_scan)) {
		void (*scan_res_handler)(struct wpa_supplicant *wpa_s,
					 struct wpa_scan_results *scan_res);

		scan_res_handler = wpa_s->scan_res_handler;
		wpa_s->scan_res_handler = NULL;
		scan_res_handler(wpa_s, scan_res);
		ret = 1;
		goto scan_work_done;
	}
    // AP 模式忽略扫描
	if (ap) {
		wpa_dbg(wpa_s, MSG_DEBUG, "Ignore scan results in AP mode");
#ifdef CONFIG_AP
		if (wpa_s->ap_iface->scan_cb)
			wpa_s->ap_iface->scan_cb(wpa_s->ap_iface);
#endif /* CONFIG_AP */
		goto scan_work_done;
	}

	wpa_dbg(wpa_s, MSG_DEBUG, "New scan results available (own=%u ext=%u)",
		wpa_s->own_scan_running,
		data ? data->scan_info.external_scan : 0);
	if (wpa_s->last_scan_req == MANUAL_SCAN_REQ &&
	    wpa_s->manual_scan_use_id && wpa_s->own_scan_running &&
	    own_request && !(data && data->scan_info.external_scan)) {
		wpa_msg_ctrl(wpa_s, MSG_INFO, WPA_EVENT_SCAN_RESULTS "id=%u",
			     wpa_s->manual_scan_id);
		wpa_s->manual_scan_use_id = 0;
	} else {
		wpa_msg_ctrl(wpa_s, MSG_INFO, WPA_EVENT_SCAN_RESULTS);
	}
	wpas_notify_scan_results(wpa_s);

	wpas_notify_scan_done(wpa_s, 1);

	if (data && data->scan_info.external_scan) {
		wpa_dbg(wpa_s, MSG_DEBUG, "Do not use results from externally requested scan operation for network selection");
		wpa_scan_results_free(scan_res);
		return 0;
	}

	if (wnm_scan_process(wpa_s, 1) > 0)
		goto scan_work_done;

	if (sme_proc_obss_scan(wpa_s) > 0)
		goto scan_work_done;

	if (own_request &&
	    wpas_beacon_rep_scan_process(wpa_s, scan_res, &data->scan_info) > 0)
		goto scan_work_done;

	if ((wpa_s->conf->ap_scan == 2 && !wpas_wps_searching(wpa_s)))
		goto scan_work_done;

	if (autoscan_notify_scan(wpa_s, scan_res))
		goto scan_work_done;

	if (wpa_s->disconnected) {
		wpa_supplicant_set_state(wpa_s, WPA_DISCONNECTED);
		goto scan_work_done;
	}

	if (!wpas_driver_bss_selection(wpa_s) &&
	    bgscan_notify_scan(wpa_s, scan_res) == 1)
		goto scan_work_done;

	wpas_wps_update_ap_info(wpa_s, scan_res);

	if (wpa_s->wpa_state >= WPA_AUTHENTICATING &&
	    wpa_s->wpa_state < WPA_COMPLETED)
		goto scan_work_done;

	wpa_scan_results_free(scan_res);

	if (own_request && wpa_s->scan_work) {
		struct wpa_radio_work *work = wpa_s->scan_work;
		wpa_s->scan_work = NULL;
		radio_work_done(work);
	}

	return wpas_select_network_from_last_scan(wpa_s, 1, own_request);

scan_work_done:
	wpa_scan_results_free(scan_res);
	if (own_request && wpa_s->scan_work) {
		struct wpa_radio_work *work = wpa_s->scan_work;
		wpa_s->scan_work = NULL;
		radio_work_done(work);
	}
	return ret;
}

// 这里对扫描结果进行处理，找到了就进行连接，没找到就继续扫描。
static int wpas_select_network_from_last_scan(struct wpa_supplicant *wpa_s,
					      int new_scan, int own_request)
{
	struct wpa_bss *selected;
	struct wpa_ssid *ssid = NULL;
	int time_to_reenable = wpas_reenabled_network_time(wpa_s);

	if (time_to_reenable > 0) {
		wpa_dbg(wpa_s, MSG_DEBUG,
			"Postpone network selection by %d seconds since all networks are disabled",
			time_to_reenable);
		eloop_cancel_timeout(wpas_network_reenabled, wpa_s, NULL);
		eloop_register_timeout(time_to_reenable, 0,
				       wpas_network_reenabled, wpa_s, NULL);
		return 0;
	}

	if (wpa_s->p2p_mgmt)
		return 0; /* no normal connection on p2p_mgmt interface */
   
	selected = wpa_supplicant_pick_network(wpa_s, &ssid);                             // 选择网络

#ifdef CONFIG_MESH
	if (wpa_s->ifmsh) {
		wpa_msg(wpa_s, MSG_INFO,
			"Avoiding join because we already joined a mesh group");
		return 0;
	}
#endif /* CONFIG_MESH */

	if (selected) {
		int skip;
		
		skip = !wpa_supplicant_need_to_roam(wpa_s, selected, ssid);        // 是否需要漫游, 后面再分析
		if (skip) {
		        
			if (new_scan)
				wpa_supplicant_rsn_preauth_scan_results(wpa_s);            // 快速漫游，预认证, 漫游才会用到
			return 0;
		}

		if (ssid != wpa_s->current_ssid &&
		    wpa_s->wpa_state >= WPA_AUTHENTICATING) {
			wpa_s->own_disconnect_req = 1;
			wpa_supplicant_deauthenticate(
				wpa_s, WLAN_REASON_DEAUTH_LEAVING);
		}
        
		if (wpa_supplicant_connect(wpa_s, selected, ssid) < 0) {                // 连接网络
			wpa_dbg(wpa_s, MSG_DEBUG, "Connect failed");
			return -1;
		}
		if (new_scan)
			wpa_supplicant_rsn_preauth_scan_results(wpa_s);
		/*
		 * Do not allow other virtual radios to trigger operations based
		 * on these scan results since we do not want them to start
		 * other associations at the same time.
		 */
		return 1;
	} else {
	    // 没找到合适的网络
		wpa_dbg(wpa_s, MSG_DEBUG, "No suitable network found");
		ssid = wpa_supplicant_pick_new_network(wpa_s);
		if (ssid) {
			wpa_dbg(wpa_s, MSG_DEBUG, "Setup a new network");
			wpa_supplicant_associate(wpa_s, NULL, ssid);
			if (new_scan)
				wpa_supplicant_rsn_preauth_scan_results(wpa_s);
		} else if (own_request) {
			/*
			 * No SSID found. If SCAN results are as a result of
			 * own scan request and not due to a scan request on
			 * another shared interface, try another scan.
			 */
			int timeout_sec = wpa_s->scan_interval;
			int timeout_usec = 0;

			......
			
            // 重新扫描
			if (wpa_supplicant_req_sched_scan(wpa_s))
				wpa_supplicant_req_new_scan(wpa_s, timeout_sec,
							    timeout_usec);

			wpa_msg_ctrl(wpa_s, MSG_INFO,
				     WPA_EVENT_NETWORK_NOT_FOUND);
		}
	}
	return 0;
}

// 选择网络
struct wpa_bss * wpa_supplicant_pick_network(struct wpa_supplicant *wpa_s,
					     struct wpa_ssid **selected_ssid)
{
	struct wpa_bss *selected = NULL;
	int prio;
	struct wpa_ssid *next_ssid = NULL;
	struct wpa_ssid *ssid;

	if (wpa_s->last_scan_res == NULL ||
	    wpa_s->last_scan_res_used == 0)
		return NULL; /* no scan results from last update */

	if (wpa_s->next_ssid) {
		/* check that next_ssid is still valid */
		for (ssid = wpa_s->conf->ssid; ssid; ssid = ssid->next) {
			if (ssid == wpa_s->next_ssid)
				break;
		}
		next_ssid = ssid;
		wpa_s->next_ssid = NULL;
	}

	while (selected == NULL) {
		for (prio = 0; prio < wpa_s->conf->num_prio; prio++) {
			if (next_ssid && next_ssid->priority ==
			    wpa_s->conf->pssid[prio]->priority) {
				selected = wpa_supplicant_select_bss(						// 选择网络
					wpa_s, next_ssid, selected_ssid, 1);
				if (selected)
					break;
			}
			selected = wpa_supplicant_select_bss(
				wpa_s, wpa_s->conf->pssid[prio],
				selected_ssid, 0);
			if (selected)
				break;
		}
        
		if (selected == NULL && wpa_s->blacklist &&						// 选择之后，没有合适的AP，清除黑名单
		    !wpa_s->countermeasures) {
			wpa_dbg(wpa_s, MSG_DEBUG, "No APs found - clear "
				"blacklist and try again");
			wpa_blacklist_clear(wpa_s);
			wpa_s->blacklist_cleared++;
		} else if (selected == NULL)
			break;
	}

	ssid = *selected_ssid;
	if (selected && ssid && ssid->mem_only_psk && !ssid->psk_set &&
	    !ssid->passphrase && !ssid->ext_psk) {
		const char *field_name, *txt = NULL;

		wpa_dbg(wpa_s, MSG_DEBUG,
			"PSK/passphrase not yet available for the selected network");

		wpas_notify_network_request(wpa_s, ssid,
					    WPA_CTRL_REQ_PSK_PASSPHRASE, NULL);

		field_name = wpa_supplicant_ctrl_req_to_string(
			WPA_CTRL_REQ_PSK_PASSPHRASE, NULL, &txt);
		if (field_name == NULL)
			return NULL;

		wpas_send_ctrl_req(wpa_s, ssid, field_name, txt);

		selected = NULL;
	}

	return selected;
}

static struct wpa_bss *
wpa_supplicant_select_bss(struct wpa_supplicant *wpa_s,
			  struct wpa_ssid *group,
			  struct wpa_ssid **selected_ssid,
			  int only_first_ssid)
{
	unsigned int i;

	if (wpa_s->current_ssid) {
		struct wpa_ssid *ssid;

		wpa_dbg(wpa_s, MSG_DEBUG,
			"Scan results matching the currently selected network");
		for (i = 0; i < wpa_s->last_scan_res_used; i++) {
			struct wpa_bss *bss = wpa_s->last_scan_res[i];

			ssid = wpa_scan_res_match(wpa_s, i, bss, group,
						  only_first_ssid, 0);
			if (ssid != wpa_s->current_ssid)
				continue;
			wpa_dbg(wpa_s, MSG_DEBUG, "%u: " MACSTR
				" freq=%d level=%d snr=%d est_throughput=%u",
				i, MAC2STR(bss->bssid), bss->freq, bss->level,
				bss->snr, bss->est_throughput);
		}
	}

	if (only_first_ssid)
		wpa_dbg(wpa_s, MSG_DEBUG, "Try to find BSS matching pre-selected network id=%d",
			group->id);
	else
		wpa_dbg(wpa_s, MSG_DEBUG, "Selecting BSS from priority group %d",
			group->priority);

	for (i = 0; i < wpa_s->last_scan_res_used; i++) {
		struct wpa_bss *bss = wpa_s->last_scan_res[i];
		*selected_ssid = wpa_scan_res_match(wpa_s, i, bss, group,			// 匹配
						    only_first_ssid, 1);
		if (!*selected_ssid)
			continue;
		wpa_dbg(wpa_s, MSG_DEBUG, "   selected BSS " MACSTR
			" ssid='%s'",
			MAC2STR(bss->bssid),
			wpa_ssid_txt(bss->ssid, bss->ssid_len));
		return bss;
	}

	return NULL;
}

// 将发过来的ssid中的数据帧和配置文件中的参数进行匹配，查看是否正常
struct wpa_ssid * wpa_scan_res_match(struct wpa_supplicant *wpa_s,
				     int i, struct wpa_bss *bss,
				     struct wpa_ssid *group,
				     int only_first_ssid, int debug_print)
{
	u8 wpa_ie_len, rsn_ie_len;
	int wpa;
	struct wpa_blacklist *e;
	const u8 *ie;
	struct wpa_ssid *ssid;
	int osen;
#ifdef CONFIG_WAPI
	u8 wapi_ie_len;
#endif
#ifdef CONFIG_MBO
	const u8 *assoc_disallow;
#endif /* CONFIG_MBO */
	// 获取vendor信息
	ie = wpa_bss_get_vendor_ie(bss, WPA_IE_VENDOR_TYPE);
	wpa_ie_len = ie ? ie[1] : 0;

	ie = wpa_bss_get_ie(bss, WLAN_EID_RSN);
	rsn_ie_len = ie ? ie[1] : 0;

	ie = wpa_bss_get_vendor_ie(bss, OSEN_IE_VENDOR_TYPE);
	osen = ie != NULL;

	if (debug_print) {
		wpa_dbg(wpa_s, MSG_DEBUG, "%d: " MACSTR
			" ssid='%s' wpa_ie_len=%u rsn_ie_len=%u caps=0x%x level=%d freq=%d %s%s%s",
			i, MAC2STR(bss->bssid),
			wpa_ssid_txt(bss->ssid, bss->ssid_len),
			wpa_ie_len, rsn_ie_len, bss->caps, bss->level,
			bss->freq,
			wpa_bss_get_vendor_ie(bss, WPS_IE_VENDOR_TYPE) ?
			" wps" : "",
			(wpa_bss_get_vendor_ie(bss, P2P_IE_VENDOR_TYPE) ||
			 wpa_bss_get_vendor_ie_beacon(bss, P2P_IE_VENDOR_TYPE))
			? " p2p" : "",
			osen ? " osen=1" : "");
	}
	
	e = wpa_blacklist_get(wpa_s, bss->bssid);
	if (e) {
		int limit = 1;
		if (wpa_supplicant_enabled_networks(wpa_s) == 1) {
			/*
			 * When only a single network is enabled, we can
			 * trigger blacklisting on the first failure. This
			 * should not be done with multiple enabled networks to
			 * avoid getting forced to move into a worse ESS on
			 * single error if there are no other BSSes of the
			 * current ESS.
			 */
			limit = 0;
		}
		if (e->count > limit) {
			if (debug_print) {
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - blacklisted (count=%d limit=%d)",
					e->count, limit);
			}
			return NULL;
		}
	}
	// 对SSID的参数进行比对，不匹配的推出
	if (bss->ssid_len == 0) {
		if (debug_print)
			wpa_dbg(wpa_s, MSG_DEBUG, "   skip - SSID not known");
		return NULL;
	}

	if (disallowed_bssid(wpa_s, bss->bssid)) {
		if (debug_print)
			wpa_dbg(wpa_s, MSG_DEBUG, "   skip - BSSID disallowed");
		return NULL;
	}

	if (disallowed_ssid(wpa_s, bss->ssid, bss->ssid_len)) {
		if (debug_print)
			wpa_dbg(wpa_s, MSG_DEBUG, "   skip - SSID disallowed");
		return NULL;
	}

	wpa = wpa_ie_len > 0 || rsn_ie_len > 0;
	......

	for (ssid = group; ssid; ssid = only_first_ssid ? NULL : ssid->pnext) {
		int check_ssid = wpa ? 1 : (ssid->ssid_len != 0);
		int res;

		if (wpas_network_disabled(wpa_s, ssid)) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG, "   skip - disabled");
			continue;
		}

		res = wpas_temp_disabled(wpa_s, ssid);
		if (res > 0) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - disabled temporarily for %d second(s)",
					res);
			continue;
		}
		
		if (ssid->bssid_set && ssid->ssid_len == 0 &&
		    os_memcmp(bss->bssid, ssid->bssid, ETH_ALEN) == 0)
			check_ssid = 0;

		if (check_ssid &&
		    (bss->ssid_len != ssid->ssid_len ||
		     os_memcmp(bss->ssid, ssid->ssid, bss->ssid_len) != 0)) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - SSID mismatch");
			continue;
		}

		if (ssid->bssid_set &&
		    os_memcmp(bss->bssid, ssid->bssid, ETH_ALEN) != 0) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - BSSID mismatch");
			continue;
		}
		// 黑名单的跳过，不进行匹配
		/* check blacklist */
		if (ssid->num_bssid_blacklist &&
		    addr_in_list(bss->bssid, ssid->bssid_blacklist,
				 ssid->num_bssid_blacklist)) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - BSSID blacklisted");
			continue;
		}

		/* if there is a whitelist, only accept those APs */
		if (ssid->num_bssid_whitelist &&
		    !addr_in_list(bss->bssid, ssid->bssid_whitelist,
				  ssid->num_bssid_whitelist)) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - BSSID not in whitelist");
			continue;
		}

		if (!wpa_supplicant_ssid_bss_match(wpa_s, ssid, bss,
						   debug_print))
			continue;
		// 判断密钥类型是否匹配
		if (!osen && !wpa &&
		    !(ssid->key_mgmt & WPA_KEY_MGMT_NONE) &&
		    !(ssid->key_mgmt & WPA_KEY_MGMT_WPS) &&
#ifdef CONFIG_WAPI
		    !(ssid->key_mgmt & WPA_KEY_MGMT_WAPI_PSK) &&
		    !(ssid->key_mgmt & WPA_KEY_MGMT_WAPI_CERT) &&
#endif
		    !(ssid->key_mgmt & WPA_KEY_MGMT_IEEE8021X_NO_WPA)) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - non-WPA network not allowed");
			continue;
		}

		if (wpa && !wpa_key_mgmt_wpa(ssid->key_mgmt) &&
		    has_wep_key(ssid)) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - ignore WPA/WPA2 AP for WEP network block");
			continue;
		}

		if ((ssid->key_mgmt & WPA_KEY_MGMT_OSEN) && !osen) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - non-OSEN network not allowed");
			continue;
		}

......

		if (ssid->mode != IEEE80211_MODE_MESH && !bss_is_ess(bss) &&
		    !bss_is_pbss(bss)) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - not ESS, PBSS, or MBSS");
			continue;
		}

		if (ssid->pbss != 2 && ssid->pbss != bss_is_pbss(bss)) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - PBSS mismatch (ssid %d bss %d)",
					ssid->pbss, bss_is_pbss(bss));
			continue;
		}

		if (!freq_allowed(ssid->freq_list, bss->freq)) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - frequency not allowed");
			continue;
		}
......

		if (!rate_match(wpa_s, bss, debug_print)) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - rate sets do not match");
			continue;
		}

#ifndef CONFIG_IBSS_RSN
		if (ssid->mode == WPAS_MODE_IBSS &&
		    !(ssid->key_mgmt & (WPA_KEY_MGMT_NONE |
					WPA_KEY_MGMT_WPA_NONE))) {
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - IBSS RSN not supported in the build");
			continue;
		}
#endif /* !CONFIG_IBSS_RSN */

......
		if (os_reltime_before(&bss->last_update, &wpa_s->scan_min_time))
		{
			struct os_reltime diff;

			os_reltime_sub(&wpa_s->scan_min_time,
				       &bss->last_update, &diff);
			if (debug_print)
				wpa_dbg(wpa_s, MSG_DEBUG,
					"   skip - scan result not recent enough (%u.%06u seconds too old)",
				(unsigned int) diff.sec,
				(unsigned int) diff.usec);
			continue;
		}

		/* Matching configuration found */
		return ssid;
	}

	/* No matching configuration found */
	return NULL;
}

// 各种认证方式定义
external\wpa_supplicant_8\src\common\defs.h
#define WPA_KEY_MGMT_IEEE8021X BIT(0)			// 802.1X认证
#define WPA_KEY_MGMT_PSK BIT(1)					// psk
#define WPA_KEY_MGMT_NONE BIT(2)				// 无密钥
#define WPA_KEY_MGMT_IEEE8021X_NO_WPA BIT(3)
#define WPA_KEY_MGMT_WPA_NONE BIT(4)
#define WPA_KEY_MGMT_FT_IEEE8021X BIT(5)		// Fast transition + 802.1X 
#define WPA_KEY_MGMT_FT_PSK BIT(6)				// Fast transition
#define WPA_KEY_MGMT_IEEE8021X_SHA256 BIT(7)
#define WPA_KEY_MGMT_PSK_SHA256 BIT(8)
#define WPA_KEY_MGMT_WPS BIT(9)
#define WPA_KEY_MGMT_SAE BIT(10)
#define WPA_KEY_MGMT_FT_SAE BIT(11)
#define WPA_KEY_MGMT_WAPI_PSK BIT(12)
#define WPA_KEY_MGMT_WAPI_CERT BIT(13)
#define WPA_KEY_MGMT_CCKM BIT(14)
#define WPA_KEY_MGMT_OSEN BIT(15)
#define WPA_KEY_MGMT_IEEE8021X_SUITE_B BIT(16)
#define WPA_KEY_MGMT_IEEE8021X_SUITE_B_192 BIT(17)
#define WPA_KEY_MGMT_FILS_SHA256 BIT(18)
#define WPA_KEY_MGMT_FILS_SHA384 BIT(19)
#define WPA_KEY_MGMT_FT_FILS_SHA256 BIT(20)
#define WPA_KEY_MGMT_FT_FILS_SHA384 BIT(21)
#define WPA_KEY_MGMT_OWE BIT(22)
#define WPA_KEY_MGMT_DPP BIT(23)
```