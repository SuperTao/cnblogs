最近开始接触django，一些基本的操作记录于此。

参考链接：

　　http://www.ziqiangxuetang.com/django/django-tutorial.html

##### django安装

sudo apt-get install python-django -y

查看是否安装成功：

```
tony@T:~$ python
Python 2.7.6 (default, Jun 22 2015, 18:00:18) 
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import django
>>> django.VERSION
(1, 10, 2, u'final', 0)
>>> django.get_version()
'1.10.2'
```

##### 创建工程

django-admin.py startproject myproject

工程名称myproject,此时工程内部的结构:

```
tree myproject
myproject/
├── manage.py
└── myproject
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

查看工程是否创建成功 

cd myproject

运行django本地的服务器

python manage.py runserver

```
Performing system checks...

System check identified no issues (0 silenced).

You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

October 21, 2016 - 08:09:25
Django version 1.10.2, using settings 'myproject.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

```
默认8000端口
当提示端口被占用的时候，可以用其它端口：
python manage.py runserver 8001
python manage.py runserver 9999
 
#监听所有可用 ip （电脑可能有一个或多个内网ip，一个或多个外网ip，即有多个ip地址）
python manage.py runserver 0.0.0.0:8000
# 如果是外网或者局域网电脑上可以用其它电脑查看开发服务器
# 访问对应的 ip加端口，比如 http://172.16.20.2:8000
```

登录网页查看 

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161021172131420-1957104100.png)

##### 工程内创建app

在有manage.py文件目录中运行。

python manage.py startapp myapp

或

django-admin.py startapp myapp

##### 定义视图处理函数

vi myapp/views.py

```
from django.shortcuts import render

from django.http import HttpResponse
# Create your views here.

def index(request):
    return HttpResponse("hello world")
```

##### 定义视图访问的方法

在浏览器中输入什么内容，调用什么函数进行处理

vi myproject/urls.py

```
from django.conf.urls import url
from django.contrib import admin

from myapp import views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^myapp/', views.index),
]
```

python manage.py runserver

浏览器查看

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161021172309732-1853316233.png)

##### 添加模板 

使用HttpResponse函数只是但存的现实文本，如果要使得显示更加丰富，就需要使用模板。

添加app到project的settings.py中

新建的 app 如果不加到 INSTALL_APPS 中的话, django 就不能自动找到app中的模板文件(app-name/templates/下的文件)和静态文件(app-name/static/中的文件)

vi myproject/settings.py

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
 
    'myapp',
]
```

使用模板进行渲染

vi myproject/myapp/views
```
from django.shortcuts import render

from django.http import HttpResponse
# Create your views here.

def index(request):
    return render(request, 'home.html')
```

创建模板文件
mkdir myapp/templates

vi myapp/templates/home.html
```
<!DOCTYPE html>
<html>
<head>
<title>welcome</title>
</head>
<body>
hello welcome
</body> 
</html>
```
登录浏览器查看

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161021172431732-1059605354.png)

##### 传递参数给模板文件

```
from django.shortcuts import render
 
from django.http import HttpResponse
# Create your views here.
 
def index(request):
    name = 'tony'
    return render(request, 'home.html', { 'usr' : name })
```

vi myapp/templates/home.html
```
<!DOCTYPE html>
<html>
<head>
<title>welcome</title>
</head>
<body>
hello {{ usr }}
</body> 
</html>
```
登录查看

![](http://images2015.cnblogs.com/blog/745188/201610/745188-20161021172455388-557964911.png)



Tony Liu

2016-10-21, Shenzhen