在android源码中编译app通过，运行时出现错误:

`"android.uid.systemandroid.view.InflateException: Binary XML file line #7: Error inflating class android.webkit.WebView`

原因是在AndroidManifest.xml中添加了`android:sharedUserId="android.uid.system"`可以共享系统权限.所以导致报错。
	
将这个删除之后，问题解决。
	
由于要设置网络，如果没有系统权限，设置会失败。根据Logcat信息，添加对应的网络权限，如下。

```
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />```
```

Tony Liu 

2017-4-12, Shenzhen