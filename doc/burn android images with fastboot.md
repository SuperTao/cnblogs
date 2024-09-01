在qualcomm平台，使用fastboot烧写系统镜像。烧写方法记录于此。

* Burn emmc_appsboot.mbn

```
adb reboot bootloader		# 重启到bootloader
fastboot flash aboot <path to emmc_appsboot.mbn >		# 烧写指定分区
fastboot reboot				# 重启
```

* Burn boot.img

```
adb reboot bootloader
fastboot flash boot <path to boot.img>
fastboot reboot
```

* Burn system.img

```
adb reboot bootloader
fastboot flash system <path to system.img>
fastboot reboot
```

* Burn userdata.img

```
adb reboot bootloader
fastboot flash userdata <path to userdata.img>
fastboot reboot
```

* Burn recovery.img

```
adb reboot bootloader
fastboot flash recovery <path to recovery.img>
fastboot reboot
```

Tony Liu

2017-4-28, Shenzhen