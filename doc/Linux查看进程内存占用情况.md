https://www.cnblogs.com/zhjh256/p/9922176.html

https://cnblogs.com/arnoldlu/p/12162860.html

```
1. 通过meminfo查看
查看某个进程的内存占用。
首先清除缓存
echo 3 > /proc/sys/vm/drop_caches
读取内存
cat /proc/meminfo
启动进程
xxxx
再次清楚缓存
echo 3 > /proc/sys/vm/drop_caches
读取内存
cat /proc/meminfo

查看两次MemFree的差值。

2. 通过/proc/pid/status查看
cat /proc/pid/status也可以查看内存使用情况。
VmHWM: 47940 kB-----------------------------RSS峰值。
VmRSS: 47940 kB-----------------------------RSS实际使用量=RSSAnon+RssFile+RssShmem。
RssAnon: 38700 kB
RssFile: 9240 kB

RssFile是库代码映射，是多个进程公用，所以如果以VmRSS的值作为参考，获得的内存会比较大。
```