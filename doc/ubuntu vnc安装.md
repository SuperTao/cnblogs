VNC（Virtual Network Computing），为一种使用RFB协议的屏幕画面分享及远程操作软件。此软件借由网络，可发送键盘与鼠标的动作及即时的屏幕画面。

##### 1. 安装vnc服务器

　　sudo apt-get install vnc4server

##### 2. 给当前用户设置vnc登录密码

```
tony@T:~$ vncpasswd
Password:
Verify:
```

##### 3. 设置远程连接显示方式

　　tony@T:~$ sudo vi ~/.vnc/xstartup

　　最后一行是显示方式，可不修改。

　　不更改的话，登录vnc的时候不能显示桌面。修改可执行以下命令：

　　sudo cp /etc/X11/Xsession ~/.vnc/xstartup

##### 4. 启动vnc server

　　vncserver :1

　　或者

　　vncserver –geometry 1280x800 :1

```
　　vnc 服务监听3个TCP端口

　　RFB(Remote FrameBuffer)协议 默认端口 : 5900+显示器号

　　HTTP协议默认端口 : 5800+显示器号

　　X协议 默认端口 : 6000+显示器号

　　–geometry 1280x800 表示vnc登录时，显示的尺寸。

　　":1"表示vnc的显示器号，如果有多个用户，依次增加。

　　并且连接端口号是5901。因此设置防火墙时，设置的端口号应该是5901。
```

##### 5. 关闭vnc server：

　　vncserver –kill :1

##### 6. 设置开机自启动
```
    vi /etc/rc.local
	添加以下内容：
    su tony -C '/usr/bin/vncserver -name tony -depth 16 -geometry 1280x800 :1'
```
##### 6. ubuntu连接vncserver

- 安装vnc客户端

　　sudo apt-get install xvnc4viewer

- 登录vnc：

　　vncviewer 192.168.1.107:1

##### 7. windows连接

　　realvnc下载地址：

　　　　https://www.realvnc.com/download/viewer/

![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160805223756184-1342733751.png)

##### Author

Tony Liu

2016-8-5， Shenzhen