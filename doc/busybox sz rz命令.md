之前板子和电脑之间传送文件的时候都是通过tftp网络下载。所以找了一下在文件系统中使用串口上传文件的方法。

rz和sz命令使用zmoderm协议,SecureCRT也用提供这个命令的支持。由于是串口，所以速度不是很快，但是对于传送小文件来说已经足够。

#### 参考链接：

　　http://blog.csdn.net/lioncode/article/details/7921525

　　http://blog.csdn.net/ljxfblog/article/details/38396421

#### 下载地址

　　https://ohse.de/uwe/software/lrzsz.html

#### 交叉编译：

```
tar zxvf lrzsz-0.12.20.tar.gz

cd lrzsz-0.12.20/

./configure

make CC=arm-linux-gcc

将生成的文件在src目录。

file查看文件的格式

Qt@tony:~/tool/lrzsz-0.12.20/src$ file lrz
lrz: ELF 32-bit LSB executable, ARM, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.31, not stripped

将src/lsz, src/lrz添加到板子上/bin目录，并重命名为sz何rz即可。
```

#### 使用方法：

1.上传下载的默认目录设置 

SecureCRT软件 -> Options -> session options -> X/Y/Zmodem

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161026104426671-251147601.png)


2.发送文件到电脑

file是文件名

```
sz file
root@freescale ~$ sz file 
rz
Starting zmodem transfer.  Press Ctrl+C to cancel.
Transferring file...
  100%     112 bytes  112 bytes/sec 00:00:01       0 Errors  
```

3.接收文件 

运行rz命令，secureCRT弹出选框，选择文件进行接收。接收之前，需要先删除本地的文件，否者接收会失败。

```
root@freescale ~$ rz
rz waiting to receive.
Starting zmodem transfer.  Press Ctrl+C to cancel.
Transferring file...
  100%     112 bytes  112 bytes/sec 00:00:01       0 Errors
```
![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161026104457500-490571108.png)


##### Author

Tony Liu

2016-10-26, Shenzhen