更改ubuntu配置文件出错了，只能进入单用户模式改回来。

转载至：

　　http://blog.csdn.net/lfyaa/article/details/9899497

1、重启ubuntu，VMware启动完毕后，随即长按shirft进入grub菜单；

2、选择recovery mode，按"e"键进入编辑页面；如下

3、将ro recovery nomodeset替换为rw single init=/bin/bash

4、按ctrl+x进入单用户模式，当前用户即为root

5、按ctrl+alt+del重启