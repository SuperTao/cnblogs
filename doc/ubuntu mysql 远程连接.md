最近需要远程连接mysql服务器，先进行简单的测试，过程记录于此。

#### 参考链接：

　　http://blog.chinaunix.net/uid-28458801-id-3445261.html

　　http://stackoverflow.com/questions/1420839/cant-connect-to-mysql-server-error-111

本机ip： 192.168.1.248

远程ip： 192.168.1.249

#### mysql服务器配置

```
# 1.更改配置文件

#因为远程mysql服务器,设置只监听本地的端口,更改mysql配置文件,注释掉下面的一行。打开远程服务器的配置文件.
sudo vi /etc/mysql/my.cnf # 注释下面一行。
# bind-address = 127.0.0.1
# 如果没注释上面的一行，客户端连接时，会出现如下错误。
ERROR 2003 (HY000): Can't connect to MySQL server on '192.168.1.249' (111)

# 2. 打开mysql服务
sudo service /etc/init.d/mysql start

# 3. 配置mysql服务器
qt@tony:~/Desktop$ mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 36
Server version: 5.5.53-0ubuntu0.12.04.1 (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

# 允许用户的ip是192.168.1.248,使用用户名为 tony， 密码是 password进行登录。

mysql> grant all PRIVILEGES on *.* to tony@'192.168.1.248' identified by 'password';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> quit
Bye

```

#### mysql客户端连接

```
# 连接出现错误。
# error1:
ERROR 1045 (28000): Access denied for user 'tony'@'aplex-2.local' (using password: YES)
# 我这边得情况是本地的ip没有配置，从新配置ip.
sudo ifconfig eth0 192.168.1.248.

# 连接
# 以服务器设置的用户名登录
Qt@tony:~$ mysql -h 192.168.1.248 -u tony -p
Enter password: 
# 或者以root用户登录
Qt@tony:~$ mysql -h 192.168.1.249 -u tony -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 37
Server version: 5.5.53-0ubuntu0.12.04.1 (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

```

Tony Liu

2016-11-4, Shenzhen