使用android xUtils框架，进行http的get和post验证。

#### 参考链接：

　　https://github.com/wyouflf/xUtils3

　　http://blog.csdn.net/bangyiqing/article/details/39552163

#### GET
```
    //访问局域网的django
    RequestParams params = new RequestParams("http://192.168.1.249:8000/");
             /* 添加get请求参数,addQueryStringParameter()函数。
              * 请求的参数以键值对的形式添加在url的后面。例如：
              * http://192.168.1.249:8000/?onoff=100
              */
             params.addQueryStringParameter("onoff", "100");
             //发送get请求
             x.http().get(params, new Callback.CommonCallback<String>(){
                 @Override
                 public void onSuccess(String result) {
                     Toast.makeText(x.app(), result, Toast.LENGTH_LONG).show();
                 }

                 @Override
                 public void onError(Throwable ex, boolean isOnCallback) {
                     Toast.makeText(x.app(), ex.getMessage(), Toast.LENGTH_LONG).show();
                 }

                 @Override
                 public void onCancelled(CancelledException cex) {
                     Toast.makeText(x.app(), "cancelled", Toast.LENGTH_LONG).show();
                 }

                 @Override
                 public void onFinished() {
                     Toast.makeText(x.app(), "Finish", Toast.LENGTH_LONG).show();
                 }
             });

         }
```

#### POST

```
    RequestParams params = new RequestParams("http://192.168.1.249:8000/");
             /* post和get基本上类似，但是添加参数要使用，addBodyParameter。
              * post请求参数, 添加的参数是在http的内容中，而不是url中。
              */
             params.addBodyParameter("onoff", "103");
                 x.http().post(params, new Callback.CommonCallback<String>(){
                 @Override
                 public void onSuccess(String result) {
                     Toast.makeText(x.app(), result, Toast.LENGTH_LONG).show();
                 }

                 @Override
                 public void onError(Throwable ex, boolean isOnCallback) {
                     Toast.makeText(x.app(), ex.getMessage(), Toast.LENGTH_LONG).show();
                 }

                 @Override
                 public void onCancelled(CancelledException cex) {
                     Toast.makeText(x.app(), "cancelled", Toast.LENGTH_LONG).show();
                 }

                 @Override
                 public void onFinished() {
                     Toast.makeText(x.app(), "Finish", Toast.LENGTH_LONG).show();
                 }
             });

         }
```

Tony Liu 

2016-11-5, Shenzhen