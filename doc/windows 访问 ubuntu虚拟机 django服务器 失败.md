```
配置ubuntu配置成桥接，在ubuntu虚拟机中运行django.py开发服务器。windows访问django失败。

虚拟机运行：

python manage.py runserver 0.0.0.0:8000

在windows上浏览器进行访问,访问失败。

虚拟机错误提示：

DisallowedHost: Invalid HTTP_HOST header: '192.168.1.249:8000'. You may need to add u'192.168.1.249' to ALLOWED_HOSTS.
[02/Nov/2016 02:16:20] "GET / HTTP/1.1" 500 59

按照提示更改：

打开project的settings.py,添加虚拟机的ip地址。如下。

ALLOWED_HOSTS = [u'192.168.1.249']

windows再次访问，成功。
```
Tony Liu

2016-11-2, Shenzhen