#### 安装 

　　1.安装

```
　　sudo apt-get install vsftpd
```

　　2 查看安装结果


　　安装完毕，检查vsftpd进程是否已启动，可以查看进程或者查看监听端口
```
　　ps  -eaf|grep  vsftpd
　　tony@T:~$ ps -eaf | grep vsftpd
　　root      2244     1  0 21:08 ?        00:00:00 /usr/sbin/vsftpd
　　tony      2408  2104  0 21:11 pts/0    00:00:00 grep --color=auto vsftpd

　　netstat -tnl | grep :21

　　tony@T:~$ netstat -tnl | grep :21
　　tcp        0      0 0.0.0.0:21              0.0.0.0:*               LISTEN  

　　端口21正在被监听
```

　　3. 尝试匿名用户登录 

　　安装完后不用做任何配置既可用匿名方式进行访问,默认的ftp文件夹为/srv/ftp。

　　登录可以在浏览器中，文件夹输入框中以及类型windows/Linux终端中进行测试。

　　建议使用windows/Linux终端测试，失败了会有错误输出，可以根据错误进行调试。 

　　更改配置文件vsftpd.conf

```
　　anonymous_enable=YES
```

　　匿名登录,用户名称ftp,密码不输入，直接回车

```
　　C:\Users\Tony>ftp 192.168.1.108
　　连接到 192.168.1.108。
　　220 (vsFTPd 2.2.2)
　　用户(192.168.1.108:(none)): ftp
　　331 Please specify the password.
　　密码:
　　230 Login successful.
```
　　登录到的目录就是/srv/ftp目录，在里面的文件就能够查看到

　　4. 使用ubuntu的账户登录

　　此时就访问ftp账户，默认是linux的账户,使用cmd中登录。 

　　需要注意的是，ftp服务器端文件路径是用户的家目录

```
　　C:\Users\Tony>ftp 192.168.1.108
　　连接到 192.168.1.108。
　　220 (vsFTPd 2.2.2)
　　用户(192.168.1.108:(none)): tony
　　331 Please specify the password.
　　密码:
　　230 Login successful.
```

　　5. 在ubuntu中，vsftpd的主要配置文件分布如下：

```
　　/etc/vsftpd.conf    	vsftpd服务器的配置文件
　　/usr/sbin/vsftpd    	vsftpd服务器的进程文件
　　/etc/pam.d/vsftpd   	vsftpd服务器的PAM接口配置文件
　　/var/ftp            	vsftpd服务器匿名用户的工作目录
```

　　6. vsftpd的开始、关闭和重启

```
　　sudo /etc/init.d/vsftpd  start   #开始
　　sudo /etc/init.d/vsftpd  stop    #关闭
　　sudo /etc/init.d/vsftpd  restart #重启
```

#### 创建虚拟账户

　　1．安装生成虚拟帐号数据库工具db

　　sudo apt-get install db4.8-util

　　2. 更改vsftp.conf的配置

　　修改配置之前，先备份当前配置

```
　　sudo cp /etc/vsftp.conf /etc/vsftp.conf.old
```

　　配置的服务器中 vsftpd.conf的参数使用如下：

```
　　listen=YES
　　#listen_ipv6=YES
　　anonymous_enable=NO //允许匿名用户访问，若禁止使用NO
　　local_enable=YES    //允许本地用户访问，若禁止则使用NO
　　write_enable=YES //表示是否允许本地用户有上传权限的，YES表示可以，NO表示禁止，也取决于客户端连接时使用的客户端工具
　　#local_umask=022 //设置本地用户上传建立文件时的权限掩码
　　#anon_upload_enable=YES //匿名用户上传文件使能
　　#anon_mkdir_write_enable=YES //匿名用户可以创建目录
　　dirmessage_enable=YES //用户切换进入目录时显示.message（如果存在）文件的内容
　　message_file=Welcome
　　xferlog_enable=YES      //是否开启传输日志的
　　connect_from_port_20=YES ////连接控制端口为20
　　chown_uploads=YES
　　chown_username=ftp
　　chroot_local_user=YES //所有的本地用户都被锁定在家目录下
　　chroot_list_enable=YES
　　chroot_list_file=/etc/vsftpd/vsftpd.chroot_list
　　xferlog_file=/var/log/vsftpd.log
　　xferlog_std_format=YES
　　idle_session_timeout=600
　　data_connection_timeout=120 //#数据连接的超时时间
　　#nopriv_user=ftpsecure
　　#async_abor_enable=YES
　　ascii_upload_enable=YES
　　ascii_download_enable=YES
　　ftpd_banner=Welcome to blah FTP service. //login欢迎信息
　　#deny_email_enable=YES
　　max_clients=10
　　max_per_ip=5
　　local_max_rate=256000
　　#hide_ids=YES
　　idle_session_timeout=3000
　　guest_enable=YES
　　guest_username=ftp
　　user_config_dir=/etc/vsftpd/vsftpd_user_conf
　　pam_service_name=vsftpd.vu
```

　　3. 建立各账户的home目录

　　在/home目录建立ftp的账户ftpdir目录
```
    cd /home/ftp/
　　sudo  mkdir  ftpdir
```

　　然后在ftpdir目录下创建admin, tony目录

```
　　cd  ftpdir
　　sudo  mkdir admin tony 
```

　　4.为虚拟用户创建本地系统用户

　　虚拟用户家目录为/home/ftpdir, 用户登录终端设为/bin/false(即使之不能登录系统)

```
　　sudo useradd ftp -d /home/ftpdir -s /bin/false
　　sudo chown -R ftp:ftp /home/ftpdir
```

　　5. 创建虚拟用户数据库

　　新建loguser.txt文件，

　　sudo vi /home/loguser.txt

　　里面输入虚拟用户名和密码，格式如下
```
　　admin
　　admin
　　tony
　　tony
```

　　注意不要多空格和空行，其中奇数行为用户名，偶数行为密码。

　　最后一行需要回车（否则建立数据库文件时无法识别最后一行，导致报奇数行错误）。

```
　　新建一个文件夹/etc/vsftpd，放置配置文件
　　sudo  mkdir  /etc/vsftpd
　　然后执行
　　sudo db4.8_load -T -t hash -f /home/loguser.txt /etc/vsftpd/vsftpd_login.db
　　最后设置一下数据库文件的访问权限
　　sudo chmod 600 /etc/vsftpd/vsftpd_login.db
```

　　6. 配置PAM文件
```
　　新建/etc/pam.d/vsftpd.vu，并编辑，
　　sudo touch /etc/pam.d/vsftpd.vu
　　sudo vi /etc/pam.d/vsftpd.vu
　　输入内容如下：
　　auth  required  /lib/security/pam_userdb.so  db=/etc/vsftpd/vsftpd_login
　　account  required  /lib/security/pam_userdb.so  db=/etc/vsftpd/vsftpd_login
```

　　7．新建etc/vsftpd /vsftpd_user_conf文件夹
```
　　sudo  mkdir  /etc/vsftpd/vsftpd_user_conf
　　现在，我们要把各个用户的配置文件放到目录/etc/vsftpd/vsftpd_user_conf中
　　cd  /etc/vsftpd/vsftpd_user_conf
　　sudo touch admin tony
```

　　8. 配置admin虚拟用户 

　　权限：上传，下载删除，重命名.

```
　　sudo vi /etc/vsftpd/vsftpd_user_conf/admin
　　里面添加
　　write_enable=YES
　　anon_world_readable_only=NO
　　anon_upload_enable=YES
　　anon_mkdir_write_enable=YES
　　anon_other_write_enable=YES
　　local_root=/home/ftpdir
```
　　9. 配置tony虚拟用户 

　　权限：上传，下载 

```
　　sudo vi /etc/vsftpd/vsftpd_user_conf/tony
　　里面添加
　　write_enable=YES
　　anon_world_readable_only=NO
　　anon_upload_enable=YES
　　anon_mkdir_write_enable=NO
　　anon_other_write_enable=NO
　　local_root=/home/ftpdir/tony
```

　　10. 新建/etc/vsftpd/vsftpd.chroot_list
```
　　sudo touch /etc/vsftpd/vsftpd.chroot_list
　　sudo vi /etc/vsftpd/vsftpd.chroot_list
　　里面添加
　　admin
```

#### Author

Tony Liu

2016-8-2, Shenzhen