设置wifi的RSSI达到阈值之后自动断开。

wifi状态改变，会更新状态栏，在状态栏中更改。

```
--- a/packages/SystemUI/src/com/android/systemui/statusbar/policy/WifiSignalController.java
+++ b/packages/SystemUI/src/com/android/systemui/statusbar/policy/WifiSignalController.java
@@ -106,6 +106,13 @@ public class WifiSignalController extends
         mCurrentState.ssid = mWifiTracker.ssid;
         mCurrentState.rssi = mWifiTracker.rssi;
         mCurrentState.level = mWifiTracker.level;
+               if (mCurrentState.connected && mCurrentState.rssi < -80)
+                       mWifiManager.disconnect();
         notifyListenersIfNecessary();
     } 
```

2018-6-14