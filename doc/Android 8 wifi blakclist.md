在连接wifi的时候，认证或者关联失败，有时会加入黑名单中。记录wpa_supplicant中blacklist的原理。

分析可以看到，如果是机器自己断开，是不会把AP加入黑名单的，只有AP侧出了问题，才有可能加入黑名单。
```
首先看一下blacklist的结构体。
external\wpa_supplicant_8\wpa_supplicant\blacklist.h
struct wpa_blacklist {
    struct wpa_blacklist *next;
    u8 bssid[ETH_ALEN];
    int count;
};

external\wpa_supplicant_8\wpa_supplicant\blacklist.c
/**
 * wpa_blacklist_get - Get the blacklist entry for a BSSID
 * @wpa_s: Pointer to wpa_supplicant data
 * @bssid: BSSID
 * Returns: Matching blacklist entry for the BSSID or %NULL if not found
 */
struct wpa_blacklist * wpa_blacklist_get(struct wpa_supplicant *wpa_s,
					 const u8 *bssid)
{
	struct wpa_blacklist *e;

	if (wpa_s == NULL || bssid == NULL)
		return NULL;

	e = wpa_s->blacklist;
	while (e) {
		if (os_memcmp(e->bssid, bssid, ETH_ALEN) == 0)
			return e;
		e = e->next;
	}

	return NULL;
}

/**
 * wpa_blacklist_add - Add an BSSID to the blacklist
 * @wpa_s: Pointer to wpa_supplicant data
 * @bssid: BSSID to be added to the blacklist
 * Returns: Current blacklist count on success, -1 on failure
 *
 * This function adds the specified BSSID to the blacklist or increases the
 * blacklist count if the BSSID was already listed. It should be called when
 * an association attempt fails either due to the selected BSS rejecting
 * association or due to timeout.
 *
 * This blacklist is used to force %wpa_supplicant to go through all available
 * BSSes before retrying to associate with an BSS that rejected or timed out
 * association. It does not prevent the listed BSS from being used; it only
 * changes the order in which they are tried.
 */
int wpa_blacklist_add(struct wpa_supplicant *wpa_s, const u8 *bssid)
{
	struct wpa_blacklist *e;

	if (wpa_s == NULL || bssid == NULL)
		return -1;

	e = wpa_blacklist_get(wpa_s, bssid);
	if (e) {			// 如果blacklist中已经存在这个ssid，，count++
		e->count++;
		wpa_printf(MSG_DEBUG, "BSSID " MACSTR " blacklist count "
			   "incremented to %d",
			   MAC2STR(bssid), e->count);
		return e->count;
	}
	// 不存在就添加到blacklist链表中
	e = os_zalloc(sizeof(*e));
	if (e == NULL)
		return -1;
	// 这个插入方式应该是从链表头插入
	os_memcpy(e->bssid, bssid, ETH_ALEN);
	e->count = 1;
	e->next = wpa_s->blacklist;
	wpa_s->blacklist = e;
	wpa_printf(MSG_DEBUG, "Added BSSID " MACSTR " into blacklist",
		   MAC2STR(bssid));

	return e->count;
}


/**
 * wpa_blacklist_del - Remove an BSSID from the blacklist
 * @wpa_s: Pointer to wpa_supplicant data
 * @bssid: BSSID to be removed from the blacklist
 * Returns: 0 on success, -1 on failure
 */
int wpa_blacklist_del(struct wpa_supplicant *wpa_s, const u8 *bssid)
{
	struct wpa_blacklist *e, *prev = NULL;

	if (wpa_s == NULL || bssid == NULL)
		return -1;

	e = wpa_s->blacklist;
	while (e) {
		if (os_memcmp(e->bssid, bssid, ETH_ALEN) == 0) {
			if (prev == NULL) {
				wpa_s->blacklist = e->next;
			} else {
				prev->next = e->next;
			}
			wpa_printf(MSG_DEBUG, "Removed BSSID " MACSTR " from "
				   "blacklist", MAC2STR(bssid));
			os_free(e);
			return 0;
		}
		prev = e;
		e = e->next;
	}
	return -1;
}

/**
 * wpa_blacklist_clear - Clear the blacklist of all entries
 * @wpa_s: Pointer to wpa_supplicant data
 */
void wpa_blacklist_clear(struct wpa_supplicant *wpa_s)
{
	struct wpa_blacklist *e, *prev;
	int max_count = 0;

	e = wpa_s->blacklist;
	wpa_s->blacklist = NULL;
	while (e) {
		// 获取count的最大值
		if (e->count > max_count)
			max_count = e->count;
		prev = e;
		e = e->next;
		wpa_printf(MSG_DEBUG, "Removed BSSID " MACSTR " from "
			   "blacklist (clear)", MAC2STR(prev->bssid));
		os_free(prev);
	}
	// 临时的计数，判断删除最后一条blacklist是否有失败。
	wpa_s->extra_blacklist_count += max_count;
}

wpa_supplicant.c
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
	// 如果断开连接是机器自己产生的，就不会把AP加入到blacklist中，直接返回。
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
	// 将BSSID加入blacklist中，加速扫描其他可以连接并且关联过的AP。
	// 并且对这个BSSID的扫描间隔增加延时，避免连续多次扫描。
	/*
	 * Add the failed BSSID into the blacklist and speed up next scan
	 * attempt if there could be other APs that could accept association.
	 * The current blacklist count indicates how many times we have tried
	 * connecting to this AP and multiple attempts mean that other APs are
	 * either not available or has already been tried, so that we can start
	 * increasing the delay here to avoid constant scanning.
	 */
	// 加入黑名单，并获取在黑名单的计数
	count = wpa_blacklist_add(wpa_s, bssid);
	if (count == 1 && wpa_s->current_bss) {
	//这个BSS以前没有在blacklist中，如果在同一个ESS中有另外的BSS，
	// 那么另外一个BSS我们也要进行尝试
	// 并且在这之前，对这个BSS进行在进行一个尝试，不过还不行，count再加一。
		/*
		 * This BSS was not in the blacklist before. If there is
		 * another BSS available for the same ESS, we should try that
		 * next. Otherwise, we may as well try this one once more
		 * before allowing other, likely worse, ESSes to be considered.
		 */
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
			wpa_s->next_scan_freqs = freqs;
		}
	}

	/*
	 * Add previous failure count in case the temporary blacklist was
	 * cleared due to no other BSSes being available.
	 */
	count += wpa_s->extra_blacklist_count;
        // count大于3，直接设置认证失败。
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
	// 进行下一次扫描，后面两个参数是扫描的时间
	/*
	 * TODO: if more than one possible AP is available in scan results,
	 * could try the other ones before requesting a new scan.
	 */
	 // 例如timeout=10000, 那么再次扫描的时间10000/1000+1000*(10000%1000) = 10s
	wpa_supplicant_req_scan(wpa_s, timeout / 1000,
				1000 * (timeout % 1000));
}
```