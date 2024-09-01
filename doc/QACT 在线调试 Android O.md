使用QACT调试Android O的音频时，不能在线调试，但是使用Android N却可以在线调试。

解决方法

```
1. adb root
2. adb remount
3. adb shell
4. chmod 777 /dev/diag
5. setenforce 0
6. getenforce (jutst to confirm it is "Permissive")
7. ps -A | grep “audio”
	i.e.
		audioserver 3669 1 39312 10644 binder_thread_read 0 S android.hardware.audio@2.0-service
		audioserver 3670 1 51620 11912 binder_thread_read 0 S audioserver 

    Please look for the process ID for android.hardware.audio@2.0-service and kill it.

8. kill 3669
```
按照上面的操作，使用QACT_v7.1.8可以连接Android O在线调试，但是使用QACT_v7.0.7连接Android O失败。

Tony Liu

2018-3-14