#### 安装samba

`subo apt install samba`

#### 现有账户/home/tao添加到samba

`sudo smbpasswd -a tao`

```
  New SMB password:
  
  Retype new SMB password:
```

`sudo smbpasswd -e tao`
 
```
  Enabled user tao.
```

#### 更改配置 

`/etc/samba/smb.conf`

```
[tao]
   path = /home/tao
   browseable = yes 
   guest ok = no
   read only = no  
   writeable = yes 
```   
#### 重启服务

`sudo service smbd restart`

#### windows远程访问

* windows开启samba服务

* 文件夹里直接访问\\192.168.1.xxx\tao

* windows映射网络驱动器，就会生成一个盘符。
