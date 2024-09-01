使用QACT调试音频，首先安装QPST，并安装对应的usb驱动，如果驱动没有安装好，有驱动精灵等软件进行安装。

QPST configure中选择对应的设备。

#### 在线调试

打开QACT，选择“Connect To Device”连接android设备。

点击DSP Calibration。

打开android设备播放音乐，就会有显示。

改变AUDIO_RX_CODEC_GAIN选框的值来设置播放器播放音乐的大小。

通过在线调试还可以知道播放音乐对应的选项是SPKR_PHONE_SPKR_MONO + AUDPROC_OFFLOAD_EFFECTS。

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171228160050366-1593866864.png)

#### 离线调试

打开对应文件，或者将连接设备调试之后的文件保存，然后通过adb将文件上传到android设备对应目录。

`/system/etc/acdbdata/QRD` 和 `/system/etc/acdbdata/MTP`

```
C:\Users\tony>adb remount		# 从新挂载，获取写权限
remount succeeded

C:\Users\tony>adb shell
tony-1:/system/etc/acdbdata/QRD # ls
ls
QRD_Bluetooth_cal.acdb QRD_Handset_cal.acdb QRD_Speaker_cal.acdb
QRD_General_cal.acdb   QRD_Hdmi_cal.acdb    msm8939-snd-card-skul
QRD_Global_cal.acdb    QRD_Headset_cal.acdb

tony-1:/system/etc/acdbdata/MTP # ls
ls
MTP_Bluetooth_cal.acdb MTP_Handset_cal.acdb MTP_Speaker_cal.acdb
MTP_General_cal.acdb   MTP_Hdmi_cal.acdb    msm8939-tapan-snd-card
MTP_Global_cal.acdb    MTP_Headset_cal.acdb
```
源码放置位置，可以通过find命令查找acdb文件。源码里的名称和文件系统中的不同，其实是一个文件，编译的时候会放到/system目录中去。

例如`vendor/qcom/proprietary/mm-audio/audcal/family-b/acdbdata/8916/QRD`
```
tony@ubuntu:~/work/asop/vendor/qcom/proprietary/mm-audio/audcal/family-b/acdbdata/8916/QRD$ ls
Bluetooth_cal.acdb  Global_cal.acdb   Hdmi_cal.acdb     msm8939-snd-card-skul  workspaceFile.qwsp
General_cal.acdb    Handset_cal.acdb  Headset_cal.acdb  Speaker_cal.acdb
```

#### 回音消除

Audio use case: Voice

Device Use case: HEADSET_MIC&HEADSET_SPKR_STEREO

选框选择 TX_VOICE_SMECNS

选择show Advanced Parameters
```
DENS_tail_portion 	代表在脉冲响应的回波尾部的能量衰减，值太小则降低噪声消除效果，太大则降低通话音质
dens_tail_alpha 	代表在脉冲响应的回声尾能量衰减，值太小会使尾音的末尾不被消除，太大则影响通话音质
dens_nl_atten		控制非线性回声抑制量，值越大，对高频回音的抵制越明显
```

![](http://images2017.cnblogs.com/blog/745188/201712/745188-20171228160118366-2062464619.png)

Tony Liu

2017-12-28