移植freetds主要是为了能够在linux下，使用C语言访问微软的sqlserver数据库。

#### 参考连接

http://blog.csdn.net/neighbor1000/article/details/8824084

http://blog.csdn.net/lovehere33/article/details/41118405

#### 在ubuntu上安装

从官网下载最新的稳定版本。

http://www.freetds.org/

以前使用旧的0.61版本，连接sqlserver服务器，速度很慢，需要几十秒，有时甚至连接不上。

后来下载稳定版本，连接速度很快。目前下载的版本是freetds-1.00.39

编译之前，查看freetds能够支持的tds协议的版本号。

`./configure --help` 查看能够支持的版本。

如下内容显示了版本号。

```
  --with-tdsver=VERSION   TDS protocol version
                          (4.2/4.6/5.0/7.0/7.1/7.2/7.3/7.4/auto) [auto]
```

运行如下命令

`./configure --prefix=/usr/local/freetds1.0 --with-tdsver=7.0 --enable-msdblib --disable-libiconv`

`make && make install`

就会在/usr/local/freetds1.0目录中安装。

测试方法
```
qt@tony:~$ cd /usr/local/freetds1.0/bin/
qt@tony:/usr/local/freetds1.0/bin$ ./tsql -H 192.168.3.126 -p 1433 -U user -P password
locale is "en_US.UTF-8"
locale charset is "UTF-8"
using default charset "UTF-8"
1> 
```

-H：sqlserver服务器的ip地址

-p: 端口号

-U: 用户名

-P：密码

查看sqlserver版本
```
1> select @@version
2> go

Microsoft SQL Server 2008 R2 (RTM) - 10.50.1600.1 (Intel X86) 
	Apr  2 2010 15:53:02 
	Copyright (c) Microsoft Corporation
	Enterprise Edition on Windows NT 5.2 <X86> (Build 3790: Service Pack 2) (Hypervisor)

(1 row affected)
1> 
```

#### arm交叉编译

```
export CC=arm-linux-gcc

export LD=arm-linux-ld

export CPP=arm-linux-cpp

./configure --host=arm-linux --prefix=/usr/local/freetds-arm --with-tdsver=7.0 --enable-msdblib --disable-libiconv

make && make install
```

之后就会在/usr/local/freetds-arm目录中有交叉编译的文件。

将其中的文件打包，放到开发板相应的/usr/local/freetds-arm目录中，移植就完成了。

#### ubuntu编译app

`gcc -o testsybase testsybase.c -L /usr/local/freetds1.0/lib/ -lsybdb -I /usr/local/freetds1.0/include/`  

testsybase.c是测试程序名称。

使用C语言编写测试app，运行时，需要先指定freetds库文件的位置。

`export LD_LIBRARY_PATH=/usr/local/freetds1.0/lib/`

#### 交叉编译app

`arm-linux-gcc -o testsybase testsybase.c -L /usr/local/freetds-arm/lib/ -lsybdb -I /usr/local/freetds-arm/include/`  

将生成的文件添加到开发板上即可。

运行时添加环境变量`export LD_LIBRARY_PATH=/usr/local/freetds7.0/lib/`

---

Tony Liu

2017-5-26, Shenzhen