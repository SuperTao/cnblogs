记录wpa_supplicant四次握手的过程。

相关log：https://www.cnblogs.com/helloworldtoyou/p/9633603.html
```
接收到第一次握手，会设置一个认证超时时间，根据情况，设置成10s或者70s。
四次握手，如果出错，将会等待这个超时时间后，才会判断认证失败。然后从新连接。

wpa_supplicant/wpa_supplicant.c
void wpa_supplicant_rx_eapol(void *ctx, const u8 *src_addr,
			     const u8 *buf, size_t len)
{
	struct wpa_supplicant *wpa_s = ctx;
	// 接受到EAPOL帧
	wpa_dbg(wpa_s, MSG_DEBUG, "RX EAPOL from " MACSTR, MAC2STR(src_addr));
	wpa_hexdump(MSG_MSGDUMP, "RX EAPOL", buf, len);

#ifdef CONFIG_TESTING_OPTIONS
	if (wpa_s->ignore_auth_resp) {
		wpa_printf(MSG_INFO, "RX EAPOL - ignore_auth_resp active!");
		return;
	}
#endif /* CONFIG_TESTING_OPTIONS */

#ifdef CONFIG_PEERKEY
	if (wpa_s->wpa_state > WPA_ASSOCIATED && wpa_s->current_ssid &&
	    wpa_s->current_ssid->peerkey &&
	    !(wpa_s->drv_flags & WPA_DRIVER_FLAGS_4WAY_HANDSHAKE) &&
	    wpa_sm_rx_eapol_peerkey(wpa_s->wpa, src_addr, buf, len) == 1) {
		wpa_dbg(wpa_s, MSG_DEBUG, "RSN: Processed PeerKey EAPOL-Key");
		return;
	}
#endif /* CONFIG_PEERKEY */

	if (wpa_s->wpa_state < WPA_ASSOCIATED ||
	    (wpa_s->last_eapol_matches_bssid &&
#ifdef CONFIG_AP
	     !wpa_s->ap_iface &&
#endif /* CONFIG_AP */
	     os_memcmp(src_addr, wpa_s->bssid, ETH_ALEN) != 0)) {
		/*
		 * There is possible race condition between receiving the
		 * association event and the EAPOL frame since they are coming
		 * through different paths from the driver. In order to avoid
		 * issues in trying to process the EAPOL frame before receiving
		 * association information, lets queue it for processing until
		 * the association event is received. This may also be needed in
		 * driver-based roaming case, so also use src_addr != BSSID as a
		 * trigger if we have previously confirmed that the
		 * Authenticator uses BSSID as the src_addr (which is not the
		 * case with wired IEEE 802.1X).
		 */
		wpa_dbg(wpa_s, MSG_DEBUG, "Not associated - Delay processing "
			"of received EAPOL frame (state=%s bssid=" MACSTR ")",
			wpa_supplicant_state_txt(wpa_s->wpa_state),
			MAC2STR(wpa_s->bssid));
		wpabuf_free(wpa_s->pending_eapol_rx);
		wpa_s->pending_eapol_rx = wpabuf_alloc_copy(buf, len);
		if (wpa_s->pending_eapol_rx) {
			os_get_reltime(&wpa_s->pending_eapol_rx_time);
			os_memcpy(wpa_s->pending_eapol_rx_src, src_addr,
				  ETH_ALEN);
		}
		return;
	}

	wpa_s->last_eapol_matches_bssid =
		os_memcmp(src_addr, wpa_s->bssid, ETH_ALEN) == 0;

#ifdef CONFIG_AP
	if (wpa_s->ap_iface) {
		wpa_supplicant_ap_rx_eapol(wpa_s, src_addr, buf, len);
		return;
	}
#endif /* CONFIG_AP */
															// 没有配置密钥，退出
	if (wpa_s->key_mgmt == WPA_KEY_MGMT_NONE) {
		wpa_dbg(wpa_s, MSG_DEBUG, "Ignored received EAPOL frame since "
			"no key management is configured");
		return;
	}
															// 第一次收到EAPOL帧，设置认证超时时间，
	if (wpa_s->eapol_received == 0 &&
	    (!(wpa_s->drv_flags & WPA_DRIVER_FLAGS_4WAY_HANDSHAKE) ||
	     !wpa_key_mgmt_wpa_psk(wpa_s->key_mgmt) ||
	     wpa_s->wpa_state != WPA_COMPLETED) &&
	    (wpa_s->current_ssid == NULL ||
	     wpa_s->current_ssid->mode != IEEE80211_MODE_IBSS)) {
		/* Timeout for completing IEEE 802.1X and WPA authentication */
		int timeout = 10;									// 认证超时时间10s

		if (wpa_key_mgmt_wpa_ieee8021x(wpa_s->key_mgmt) ||
		    wpa_s->key_mgmt == WPA_KEY_MGMT_IEEE8021X_NO_WPA ||
		    wpa_s->key_mgmt == WPA_KEY_MGMT_WPS) {
			/* Use longer timeout for IEEE 802.1X/EAP */
			timeout = 70;									// 802.1X/EAP认证超时时间设置成70s
		}
															// WPS相关设置
#ifdef CONFIG_WPS
		if (wpa_s->current_ssid && wpa_s->current_bss &&
		    (wpa_s->current_ssid->key_mgmt & WPA_KEY_MGMT_WPS) &&
		    eap_is_wps_pin_enrollee(&wpa_s->current_ssid->eap)) {
			/*
			 * Use shorter timeout if going through WPS AP iteration
			 * for PIN config method with an AP that does not
			 * advertise Selected Registrar.
			 */
			struct wpabuf *wps_ie;

			wps_ie = wpa_bss_get_vendor_ie_multi(
				wpa_s->current_bss, WPS_IE_VENDOR_TYPE);
			if (wps_ie &&
			    !wps_is_addr_authorized(wps_ie, wpa_s->own_addr, 1))
				timeout = 10;
			else {
			    /* We should apply a flexible timeout value,
                * becasue some AP which send M2D MSG
                * need more time to complete EAP auth,such as Marvell.
                * */
		        timeout = 70;
			}
			wpabuf_free(wps_ie);
		}
#endif /* CONFIG_WPS */

		wpa_supplicant_req_auth_timeout(wpa_s, timeout, 0);			// 设置认证超时，10s或者70s
	}
	wpa_s->eapol_received++;										// 接受到的EAPOL计数加一

	if (wpa_s->countermeasures) {
		wpa_msg(wpa_s, MSG_INFO, "WPA: Countermeasures - dropped "
			"EAPOL packet");
		return;
	}

#ifdef CONFIG_IBSS_RSN
	if (wpa_s->current_ssid &&
	    wpa_s->current_ssid->mode == WPAS_MODE_IBSS) {
		ibss_rsn_rx_eapol(wpa_s->ibss_rsn, src_addr, buf, len);
		return;
	}
#endif /* CONFIG_IBSS_RSN */

	/* Source address of the incoming EAPOL frame could be compared to the
	 * current BSSID. However, it is possible that a centralized
	 * Authenticator could be using another MAC address than the BSSID of
	 * an AP, so just allow any address to be used for now. The replies are
	 * still sent to the current BSSID (if available), though. */

	os_memcpy(wpa_s->last_eapol_src, src_addr, ETH_ALEN);
	if (!wpa_key_mgmt_wpa_psk(wpa_s->key_mgmt) &&
	    wpa_s->key_mgmt != WPA_KEY_MGMT_OWE &&
	    wpa_s->key_mgmt != WPA_KEY_MGMT_DPP &&
	    eapol_sm_rx_eapol(wpa_s->eapol, src_addr, buf, len) > 0)
		return;
	wpa_drv_poll(wpa_s);
	if (!(wpa_s->drv_flags & WPA_DRIVER_FLAGS_4WAY_HANDSHAKE))
		wpa_sm_rx_eapol(wpa_s->wpa, src_addr, buf, len);				// 处理接受的EAPOL帧
	else if (wpa_key_mgmt_wpa_ieee8021x(wpa_s->key_mgmt)) {
		/*
		 * Set portValid = TRUE here since we are going to skip 4-way
		 * handshake processing which would normally set portValid. We
		 * need this to allow the EAPOL state machines to be completed
		 * without going through EAPOL-Key handshake.
		 */
		eapol_sm_notify_portValid(wpa_s->eapol, TRUE);
	}
}

src/rsn_supp/wpa.c
int wpa_sm_rx_eapol(struct wpa_sm *sm, const u8 *src_addr,
            const u8 *buf, size_t len)
{
	各种帧内容判断
...
    if (key_info & WPA_KEY_INFO_KEY_TYPE) {
        if (key_info & WPA_KEY_INFO_KEY_INDEX_MASK) {
            wpa_msg(sm->ctx->msg_ctx, MSG_WARNING,
                "WPA: Ignored EAPOL-Key (Pairwise) with "
                "non-zero key index");
            goto out; 
        }    
        if (peerkey) {
            /* PeerKey 4-Way Handshake */               
            peerkey_rx_eapol_4way(sm, peerkey, key, key_info, ver, 
                          key_data, key_data_len);
        } else if (key_info & (WPA_KEY_INFO_MIC |
                       WPA_KEY_INFO_ENCR_KEY_DATA)) {
            /* 3/4 4-Way Handshake */                        // 第三次握手
            wpa_supplicant_process_3_of_4(sm, key, ver, key_data,
                              key_data_len);
        } else {
            /* 1/4 4-Way Handshake */                        // 第一次握手
            wpa_supplicant_process_1_of_4(sm, src_addr, key, 
                              ver, key_data,
                              key_data_len);
        }    
    } else if (key_info & WPA_KEY_INFO_SMK_MESSAGE) {
        /* PeerKey SMK Handshake */
        peerkey_rx_eapol_smk(sm, src_addr, key, key_data, key_data_len,
                     key_info, ver);
    } else {
        if ((mic_len && (key_info & WPA_KEY_INFO_MIC)) ||
            (!mic_len && (key_info & WPA_KEY_INFO_ENCR_KEY_DATA))) {
            /* 1/2 Group Key Handshake */
            wpa_supplicant_process_1_of_2(sm, src_addr, key, 
                              key_data, key_data_len,
                              ver);
        } else {
            wpa_msg(sm->ctx->msg_ctx, MSG_WARNING,
                "WPA: EAPOL-Key (Group) without Mic/Encr bit - "
                "dropped");
        }    
    }    
... 
}


static void wpa_supplicant_process_1_of_4(struct wpa_sm *sm,
                      const unsigned char *src_addr,
                      const struct wpa_eapol_key *key,
                      u16 ver, const u8 *key_data,
                      size_t key_data_len)
{
    struct wpa_eapol_ie_parse ie;
    struct wpa_ptk *ptk;
    int res;
    u8 *kde, *kde_buf = NULL;
    size_t kde_len;

    if (wpa_sm_get_network_ctx(sm) == NULL) {
        wpa_msg(sm->ctx->msg_ctx, MSG_WARNING, "WPA: No SSID info "
            "found (msg 1 of 4)");
        return;
    }

    wpa_sm_set_state(sm, WPA_4WAY_HANDSHAKE);
    wpa_dbg(sm->ctx->msg_ctx, MSG_DEBUG, "WPA: RX message 1 of 4-Way "
        "Handshake from " MACSTR " (ver=%d)", MAC2STR(src_addr), ver);

    os_memset(&ie, 0, sizeof(ie));

    if (sm->proto == WPA_PROTO_RSN || sm->proto == WPA_PROTO_OSEN) {
        /* RSN: msg 1/4 should contain PMKID for the selected PMK */
        wpa_hexdump(MSG_DEBUG, "RSN: msg 1/4 key data",
                key_data, key_data_len);
        if (wpa_supplicant_parse_ies(key_data, key_data_len, &ie) < 0)
            goto failed;
        if (ie.pmkid) {
            wpa_hexdump(MSG_DEBUG, "RSN: PMKID from "
                    "Authenticator", ie.pmkid, PMKID_LEN);
        }
    }

    res = wpa_supplicant_get_pmk(sm, src_addr, ie.pmkid);
    if (res == -2) {
        wpa_dbg(sm->ctx->msg_ctx, MSG_DEBUG, "RSN: Do not reply to "
            "msg 1/4 - requesting full EAP authentication");
        return;
    }
    if (res)
        goto failed;

    if (sm->renew_snonce) {
        if (random_get_bytes(sm->snonce, WPA_NONCE_LEN)) {
            wpa_msg(sm->ctx->msg_ctx, MSG_WARNING,
                "WPA: Failed to get random data for SNonce");
            goto failed;
        }
        sm->renew_snonce = 0;
        wpa_hexdump(MSG_DEBUG, "WPA: Renewed SNonce",
                sm->snonce, WPA_NONCE_LEN);
    }

    /* Calculate PTK which will be stored as a temporary PTK until it has
     * been verified when processing message 3/4. */
    ptk = &sm->tptk;
    wpa_derive_ptk(sm, src_addr, key, ptk);
    if (sm->pairwise_cipher == WPA_CIPHER_TKIP) {
        u8 buf[8];
        /* Supplicant: swap tx/rx Mic keys */
        os_memcpy(buf, &ptk->tk[16], 8);
        os_memcpy(&ptk->tk[16], &ptk->tk[24], 8);
        os_memcpy(&ptk->tk[24], buf, 8);
        os_memset(buf, 0, sizeof(buf));
    }
    sm->tptk_set = 1;

    kde = sm->assoc_wpa_ie;
    kde_len = sm->assoc_wpa_ie_len;

...
                            // 发送第二次握手的包
    if (wpa_supplicant_send_2_of_4(sm, sm->bssid, key, ver, sm->snonce,
                       kde, kde_len, ptk) < 0)
        goto failed;

    os_free(kde_buf);
    os_memcpy(sm->anonce, key->key_nonce, WPA_NONCE_LEN);
    return;

failed:
    os_free(kde_buf);
    wpa_sm_deauthenticate(sm, WLAN_REASON_UNSPECIFIED);
}
```
Liutao 

2018-11-22