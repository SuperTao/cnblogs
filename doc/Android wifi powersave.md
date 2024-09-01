使用高通平台的查看power save的功能。
```
一般是控制WCNSS_qcom_cfg.ini文件的两个参数gEnableBmps，gEnableImps。

BMPS： Beacon mode power save，连接wifi时，控制wifi功耗

IMPS ：  Idle mode power save，没连接wifi时，控制wifi功耗

使用工具测量电池电流大小。

gEnableBmps=1，亮屏电流在0.18A左右，最小0.17A，最大0.19.瞬间会有0.21的电流，0.26A。

gEnableBmps=0， 亮屏电流，0.21-0.22A。

gEnableImps=0，0.20-0.22A ，瞬间会有0.23，0.28A电流。

gEnableImps=1，0.15-0.16 , 瞬间电流最大，0.18，0.22A。

没加载wlan模块， 0.16-0.17A。
```