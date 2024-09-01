android O连接Wifi，可以上网，但是却提示无法访问网络，并且在wifi图标上有一个'x'。

从android N开始引入了监控机制，每次连接都会访问一下google的服务器，由于国内被墙，所以就会出先上面的问题。连接vpn，访问google就不会有'x'。

logcat错误：
```
03-30 00:32:37.407  1385  5819 D NetworkMonitor/NetworkAgentInfo [WIFI () - 102]: PROBE_HTTPS https://www.google.com/generate_204 Probe failed with exception java.net.SocketTimeoutException: failed to connect to www.google.com/75.126.33.156 (port 443) from /192.168.1.194 (port 41944) after 10000ms
```

对应文件`frameworks/base/services/core/java/com/android/server/connectivity/NetworkMoniter.java`

```
public class NetworkMonitor extends StateMachine {
    private static final String TAG = NetworkMonitor.class.getSimpleName();
    private static final boolean DBG  = true;
    private static final boolean VDBG = false;

    // Default configuration values for captive portal detection probes.
    // TODO: append a random length parameter to the default HTTPS url.
    // TODO: randomize browser version ids in the default User-Agent String.
    private static final String DEFAULT_HTTPS_URL     = "https://www.google.com/generate_204";
    private static final String DEFAULT_HTTP_URL      =    
            "http://connectivitycheck.gstatic.com/generate_204";
    private static final String DEFAULT_FALLBACK_URL  = "http://www.google.com/gen_204";
    private static final String DEFAULT_OTHER_FALLBACK_URLS =
            "http://play.googleapis.com/generate_204";
    private static final String DEFAULT_USER_AGENT    = "Mozilla/5.0 (X11; Linux x86_64) "
                                                      + "AppleWebKit/537.36 (KHTML, like Gecko) "
                                                      + "Chrome/60.0.3112.32 Safari/537.36";
```
将上面的DEFAULT_HTTPS_URL更改为可以访问的网站即可，例如"https//www.google.cn/generate_204"

提示不能访问的代码网络的代码：
```
packages/apps/Settings/src/com/android/settings/wifi/WifiNoInternetDialog.java:            ap.mMessage = getString(R.string.no_internet_access_text);
```

Tony Liu

2018-4-10