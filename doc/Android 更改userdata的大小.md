在使用fastboot刷userdata时，出现错误：

```
(bootloader)  flash img partition: userdata
FAILED (remote: size too large)
```

原因是分区的实际大小和定义的不一致。需要更改。

参考链接：https://www.jianshu.com/p/dd0ba4db3243

使用`ls -l /dev/block/platform/soc/by-name`查看userdata对应的是哪一个mmcblk。

然后`cat /proc/partitions`查看分区的大小，转换成bit。

更改Boardconfig.mk的`BOARD_USERDATAIMAGE_PARTITION_SIZE`即可。