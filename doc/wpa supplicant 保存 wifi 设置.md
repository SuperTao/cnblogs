wpa suppliclant使用wpa gui连接wifi后，下次开机的时,不能保存,需要从新手动进行连接。

自动保存方法:

配置文件/etc/wpa_supplicant.conf

添加 `update_config=1`

新的账户和密码就会以如下的形式保存在/etc/wpa_supplicant.conf中，下次开机启动就会读取这个配置文件。

```
network={
        ssid="CCC" 
        psk="66666666"
        proto=RSN
        key_mgmt=WPA-PSK
        pairwise=CCMP
        auth_alg=OPEN
}
```

Tony Liu

2016-12-12, Shenzhen