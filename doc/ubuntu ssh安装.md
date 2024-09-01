SSH(Secure Shell)为一项创建在应用层和传输层基础上的安全协议，为计算机上的Shell提供安全的传输和使用环境。

通过SSH可以对所有传输的数据进行加密，也能够防止DNS欺骗和IP欺骗。

##### 1. 安装ssh客户端和服务器

　　sudo apt-get install openssh-client

　　sudo apt-get install openssh-server

##### 2. 修改配置

　　ssh默认的端口号是22，可以更改。

　　sudo vim /etc/ssh/ssh_config

　　#Port 22

##### 3. 服务重启

　　sudo /etc/init.d/ssh stop

　　sudo /etc/init.d/ssh start

　　sudo /etc/init.d/ssh restart

##### 4. Linux 连接

　　ssh username@ipaddress

```
tony@T:~$ ssh tony@192.168.1.107
tony@192.168.1.107's password: 
Linux T 2.6.32-38-generic #85-Ubuntu SMP Wed Jan 25 13:59:45 UTC 2012 i686 GNU/Linux
Ubuntu 10.04.4 LTS

Welcome to Ubuntu!
 * Documentation:  https://help.ubuntu.com/

New release 'precise' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Fri Aug  5 21:18:35 2016 from 192.168.1.104
tony@T:~$ 
```

##### 5. windows连接

　　使用putty软件进行连接

![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160805212552340-908817241.png)


```
login as: tony
tony@192.168.1.107's password:
Linux T 2.6.32-38-generic #85-Ubuntu SMP Wed Jan 25 13:59:45 UTC 2012 i686 GNU/Linux
Ubuntu 10.04.4 LTS

Welcome to Ubuntu!
 * Documentation:  https://help.ubuntu.com/

New release 'precise' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Fri Aug  5 20:54:34 2016 from 192.168.1.104
tony@T:~$ ls
```

##### Author

Tony Liu

2016-8-5, Shenzhen