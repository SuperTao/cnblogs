adb shell error: more than one device and emulator

##### 本文转载出处：

　　http://blog.sina.com.cn/s/blog_7ffb8dd50100wvrb.html

##### 指定所连接的设备

　　1.获取设备列表

　　　　adb devices

　　2.指定device来执行adb shell

　　　　adb -s devicename shell

##### 重新启动服务

　　1.重启adb.exe服务

　　　　adb start-server

　　2.关闭adb.exe

　　　　如果重启没有用，就尝试关闭adb.exe。

　　　　adb kill-server

　　　　或者在任务管理器中关闭adb.ext进程。