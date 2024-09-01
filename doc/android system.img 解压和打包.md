system.img重新编译的时间太长，添加和更改的文件系统内容，往往通过对system.img加压再打包的方式。

##### 参考链接

　　http://blog.csdn.net/whu_zhangmin/article/details/32937213

##### 查看文件类型

　　file system.img

　　system.img: Linux rev 1.0 ext4 filesystem data, UUID=57f8f4bc-abf4-655f-bf67-946fc0f9f25b (extents) (large files)

##### 解压(挂载)

　　挂载到当前system目录中

　　mount -t ext4 -o loop system.img ./system/

##### 重新打包(卸载)

　　umount ./system