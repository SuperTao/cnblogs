c#定时器有很多种，本文只记录其中一种多线程定时器，详情查看参考链接。

###### 参考链接

http://www.cnblogs.com/LoveJenny/archive/2011/05/28/2053697.html

http://www.cnblogs.com/yank/archive/2007/12/03/981238.html

###### 代码
```
    #using System.Timers;

    System.Timers.Timer tmr;
    tmr = new System.Timers.Timer();
    tmr.Interval = (double)(1000);  //ms
    tmr.Elapsed += new ElapsedEventHandler(tmr_timeout);
    tmr.Enabled = true;
    tmr.AutoReset = false;      //false:只定时一次, true：重复执行
    tmr.Start();

    void tmr_timeout(object sender, ElapsedEventArgs e)
    {
        Console.WriteLine("Tick...");
    }
```

###### Author

Tony Liu

2016-7-7, Shenzhen