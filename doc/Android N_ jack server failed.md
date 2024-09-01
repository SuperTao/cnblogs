在服务器上编译Android N。出现如下错误。

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171208152802796-831437557.png)

Android N 编译时会使用到 jack server，同一台服务器上，各个用户都需要为 jack server 指定不同的端口，否则会产生端口冲突。

需要修改的文件和修改内容

更改service.port和admin.port的值。

#### 修改  ~/.jack-server/config.properties 

```
jack.server.max-jars-size=104857600
jack.server.max-service=4
jack.server.service.port=8078
jack.server.max-service.by-mem=1\=2147483648\:2\=3221225472\:3\=4294967296
jack.server.admin.port=8079
jack.server.config.version=2
jack.server.time-out=7200
```

#### 修改 ~/.jack-settings

```
# Server settings
SERVER_HOST=127.0.0.1
SERVER_PORT_SERVICE=8078
SERVER_PORT_ADMIN=8079

# Internal, do not touch
SETTING_VERSION=4
```

#### 注意

这两个文件中的 service port 和 admin port 要保持一致

其中config.properties文件是自动生成的，如果文件不存在自己手动添加的话，编译还是会出错。

需要编译源码，会先自动生成cofig.properties，然后再更改里面的内容，编译就通过了。