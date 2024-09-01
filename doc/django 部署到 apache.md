安装完django之后，每次都需要通过命令来启动启动开发服务器。虽然调试和测试方便，但只能在本地运行，并且不能承受许多用户同时使用的负载。所以需要将Django部署到生产级的服务器，这里选择apache。

##### 参考链接

　　http://www.cnblogs.com/fengzheng/p/3619406.html

　　http://www.jianshu.com/p/b40a4a12fff1

　　http://www.ziqiangxuetang.com/django/django-deploy.html

　　http://blog.chinaunix.net/uid-20940095-id-4408225.html

##### 1. ubuntu安装apache

　　sudo apt-get install apache2

##### 2. 测试apache

打开浏览器输入，127.0.0.1

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161019221706513-524853765.png)

##### 3. 建立Python与Apache的链接

```
sudo apt-get install libapache2-mod-wsgi      #Python2

sudo apt-get install libapache2-mod-wsgi-py3  #Python3
```

##### 4. 创建django工程

cd /var/www/

sudo django-admin.py startproject mysite

#### 5. 测试django工程

cd mysite

采用8000端口

sudo python manage.py runserver 8000

浏览器测试

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161019222436732-1491191447.png)


#### 6. 更改端口

sudo vi /etc/apache2/ports.conf

添加：

```
NamevirtualHost *:8888
Listen 8888
```

表示VirtualHost *:8888的虚拟主机监听8888端口

#### 7. 添加网站配置文件

sudo vi /etc/apache2/sites-available/mysite.conf

```
<VirtualHost *:8888>      
    DocumentRoot /var/www/mysite/mysite      
    <Directory /var/www/mysite/mysite>          
        Order allow,deny          
        Allow from all      
    </Directory>      
  
WSGIScriptAlias / /var/www/mysite/mysite/wsgi.py
</VirtualHost>  
```

#### 8. 更改django工程

sudo vi /var/www/mysite/mysite/wsgi.py

添加

```
import sys
sys.path.append("/var/www/mysite/")
```

#### 9. 配置生效

9.1 

　　sudo a2ensite mysite.conf

有时候需要不使能配置。

　　sudo a2dissite mysite.conf

9.2 apache服务重启

　　sudo service apache2 restart

或

　　sudo service apache2 reload

出现错误;
```
restarting web server apache2                                         [fail] 
 * The apache2 configtest failed.
Output of config test was:
AH00526: Syntax error on line 8 of /etc/apache2/sites-enabled/mysite.conf:
Invalid command 'WSGIScriptAlias', perhaps misspelled or defined by a module not included in the server configuration
Action 'configtest' failed.
The Apache error log may have more information.
tony@T:/etc/apache2/sites-available$ sudo a2enmod wsgi
ERROR: Module wsgi does not exist!
```

解决方法：

```
sudo apt-get purge libapache2-mod-wsgi

sudo apt-get install libapache2-mod-wsgi
```

##### 10. 登录测试

127.0.0.1:8888

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161019221815904-1370682358.png)

##### 11. 创建app测试

cd /var/www/mysite/

python manage.py startapp blog

sudo vi blog/views

```
from django.shortcuts import render

from django.http import HttpResponse

# Create your views here.
def index(request):
    return HttpResponse("hello world")
```
sudo vi mysite/urls.py

```
from django.conf.urls import url
from django.contrib import admin 

from blog import views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^$', views.index),
]
```
访问结果。

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161019221851045-1617274267.png)



Tony Liu

2016-10-19, Shenzhen