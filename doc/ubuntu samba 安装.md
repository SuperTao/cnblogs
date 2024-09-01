Samba是在Linux和UNIX系统上实现SMB协议的一个免费软件，是一种在局域网上共享文件和打印机的一种通信协议。

##### 1. 安装

　　sudo apt-get install samba samba-common

##### 2. 创建共享目录

　　sudo mkdir /samba/share -p

##### 3. 修改共享目录权限

　　sudo chmod 777 /samba/share/ -R

##### 4. 修改samba配置

　　sudo vi /etc/samba/smb.conf
```
#需要账号密码才能访问
security = user

#配置文件最后添加
[myshare]	
		comment = my share directory
		path = /samba/share
		browseable = yes
		writable = yes

# 注释：  [myshare]共享文件夹的显示名称
#         comment 对共享的描述
#         path 共享的目录
#         browseable 该共享可以浏览
#         writable 该共享目录可写 
```

##### 5. 创建新用户
　　sudo useradd smbuser

##### 6. 为用户创建smb密码
　　sudo smbpasswd -a smbuser

##### 7. 重启服务
　　sudo service smbd restart

##### 8. windows连接

![](http://images2015.cnblogs.com/blog/745188/201608/745188-20160803222730528-759379124.png)