参考链接:

　　http://spyker729.blogspot.com/2010/07/mkimage-command-not-found-u-boot-images.html

制作uImage的工具mkimage找不到，将编译之后的uboot/tools目录中的mkimage中复制到/usr/bin/目录即可。

或者将mkimage的路径添加到环境变量PATH中。

或者使用命令安装mkimage：`sudo apt-get install uboot-mkimage`