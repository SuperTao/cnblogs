imx6在qt中打开调试串口时，ui总是会卡死。调试串口已经被文件系统占用，而在qt的app中使用open函数却能够调用open函数，打开成功，造成ui卡死，并且调试串口也卡死。本文记录这个问题的解决方法。


```
   try {
       //查找/etc/inittab文件中，是否使用ttymxc0作为调试串口输出
       //如果是，就抛出异常，不打开串口
       ret = system("grep ttymxc0 /etc/inittab | grep  -v \"#\"");
       if (ret  == 0)
       {
           throw "Current port has been used by debug port!";
       }
       fd = open( "/dev/ttymxc0", O_RDWR|O_NDELAY);
       if (-1 == fd)
       {
           printf("open fail\n");
           ::close(fd);
           delete ui;
           return(-1);
       }
       else
       {
           printf("open ttymxc0 .....\n");
       }
   }
   catch (const char * msg){
        //显示警告信息
       QMessageBox::information(this, tr("warning"), msg);
	return 0;
   }
```

Tony Liu

2016-9-8, Shenzhen