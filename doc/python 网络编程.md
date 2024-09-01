学习python网络编程的方法，参考网上代码，运行之后，记录于此。

本文的代码来源于:

　　http://www.runoob.com/python3/python3-socket.html

##### 服务器

```
#!/usr/bin/env python

#导入模块
import socket
import sys

# 创建socket对象, 
# 套接字家族AF_INET，
# 套接字类型 TCP：SOCK_STREAM。 UDP:SOCK_DGRAM
serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 获取本地主机名, host可以是主机名或者ip地址  
host = socket.gethostname()
# host = '192.168.1.113'
# 端口号
port = 9999

# 主机与端口号绑定, 
# bind函数需要使用元组(host, port)作为参数
serversocket.bind((host, port))

# 最大连接数5
serversocket.listen(5)

while True:
    # 建立客户端连接, 会一直阻塞等待连接
    # 客户端连接成功，返回一个元组，第一个元素是socket对象，第二个元素是客户端的ip地址
	clientsocket,addr = serversocket.accept()
	print("connet address %s" % str(addr))
	msg='welcom!' + "\r\n"
    # 发送信息给客户端
	clientsocket.send(msg)
    # 关闭连接
	clientsocket.close()
```

##### 客户端
```
#!/usr/bin/env python

import socket
import sys

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# host可以是主机名或者ip地址
host = socket.gethostname()
# host = '192.168.1.113'

port = 9999

# 连接服务器，connct的参数也是一个元组
# 服务器需要打开，否则连接失败
s.connect((host, port))

# 接收
msg = s.recv(1024)

s.close()

print (msg)
```

Tony Liu

2016-8-31, Shenzhen