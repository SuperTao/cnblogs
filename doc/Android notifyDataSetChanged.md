notifyDataSetChanged()用于动态的更新ListView中的数据。最后还是会调用Adapter中的getView函数。

notifyDataSetChanged()相比于setAdapter()的区别在于，对于scrap heap中存在的view，不会再去inflate。

参考链接：

https://www.pocketdigi.com/20100827/75.html

http://blog.csdn.net/chenzhiqin20/article/details/8519714

https://francisshi.github.io/2015/08/26/Android%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B9%8BNotifyDataSetChanged/