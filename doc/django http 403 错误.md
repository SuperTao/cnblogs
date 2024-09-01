在使用android的xUtils框架提交post请求到django服务器上面，出现错误，返回Forbiddeen。解决方法记录于此。

####  参考链接

　　http://blog.csdn.net/liangpz521/article/details/8102156

#### 现象

出现Forbidden的错误，查看出错的代码。django开发服务器的错误输出。

[05/Nov/2016 13:39:42] "POST / HTTP/1.1" 403 2598

http请求，403 Forbidden 是HTTP协议中的一个HTTP状态码（Status Code），表示服务器收到请求但拒绝提供服务。因此错误出在django中。

#### 解决方法

在django工程的settings.py里面的MIDDLEWARE_CLASSES中，去掉“‘django.middleware.csrf.CsrfViewMiddleware’,”

#### Author

Tony Liu

2016-11-5, Shenzhen