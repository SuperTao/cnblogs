在app中访问串口，提示没有读写权限。本文记录解决方法。

### 调试

查看设备节点权限

```
shell@msm8909:/ # ll /dev/ttyHSL*
crw------- root     root     246,   0 1970-02-07 08:13 ttyHSL0
crw-rw---- system   system   246,   1 1970-02-07 08:11 ttyHSL1
```

更改权限

```
shell@msm8909:/ # chmod 777 /dev/ttyHSL*
shell@msm8909:/ # ll /dev/ttyHSL*                                              
crwxrwxrwx root     root     246,   0 1970-02-07 08:14 ttyHSL0
crwxrwxrwx system   system   246,   1 1970-02-07 08:11 ttyHSL1
```

ttyHSL0可以读取。

更改所属组，读取ttyHSL1还是失败。

```
shell@msm8909:/ # chown root:root /dev/ttyHSL1
shell@msm8909:/ # ll /dev/ttyHSL*
crwxrwxrwx root     root     246,   0 1970-02-07 08:15 ttyHSL0
crwxrwxrwx root     root     246,   1 1970-02-07 08:11 ttyHSL1
```


关闭seLinux

```
setenforce 0
```

读写ttyHSL1成功。

#### 永久更改

编辑device/qcom/msm8909/BoardConfig.mk

添加androidboot.selinux=permissive

```
BOARD_KERNEL_CMDLINE := console=ttyHSL0,115200,n8 androidboot.console=ttyHSL0 androidboot.selinux=permissive androidboot.hardware=qcom user_debug=31 msm_rtb.filter=0x3F ehci-hcd.park=3 androidboot.bootdevice=7824900.sdhci lpm_levels.sleep_disabled=1 earlyprintk
```

编辑device/qcom/common/rootdir/etc/ueventd.qcom.rc

```
/dev/ttyHSL0              0666   root       root
/dev/ttyHSL1              0666   root       root
```

Tony Liu

2017-4-28, Shenzhen