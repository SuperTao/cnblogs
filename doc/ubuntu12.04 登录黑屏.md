新安装的ubuntu12.04LTS，登录之后黑屏，切换到ubuntu2D能够进入UI。解决方法记录于此。

转载：

　　http://blog.csdn.net/albertsh/article/details/12302473


系统黑屏后 Ctrl+ALT+F4进入命令行模式。

```
   sudo apt-get update

   sudo apt-get install xserver-xorg-lts-quantal

   sudo dpkg-reconfigure xserver-xorg-lts-quantal

   sudo reboot
```