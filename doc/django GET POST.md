django需要读取客户端get和post请求的值。读取处理方法和异常记录于此。

#### 参考链接：

　　http://stackoverflow.com/questions/12518517/request-post-getsth-vs-request-poststh-difference


* request.GET['sth']如果'sth'不存在,将会触发KeyError异常。

* request.GET.get('sth')如果'sth'不存在,将会得到None。

POST使用的时候也是一样。

#### example
```
def index(request):
    count = 0
    if request.method == 'GET':
        try:
            t = time.objects.get(id=1)
            count = request.GET['onoff']  
            t.count = int(count)
            t.save()
        except KeyError:
            t = time.objects.get(id=1)
            count = str(t.count)
            print 'hello' 
        return render(request, 'index.html', { 'onoff': count})
    return render(request, 'index.html')
```

Tony Liu 

2016-11-5, Shenzhen