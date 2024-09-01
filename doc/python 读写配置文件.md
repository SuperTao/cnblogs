使用python读写配置文件，写个demo测试一下。

```
#!/usr/bin/env python

import os

import ConfigParser

# 如果文件不存在，就穿件文件。
if os.path.isfile(os.getcwd() + '/sql.conf') == False:
    cfg = ConfigParser.ConfigParser()
    cfg.add_section('remote')                # 添加section
    cfg.set('remote', 'ip' ,'192.168.3.123')    # 添加字段
    cfg.set('remote', 'port' ,'40949')
    cfg.set('remote', 'user' ,'tony')
    cfg.set('remote', 'password' ,'tony')
    cfg.set('remote', 'table' ,'real_tabel')

    cfg.add_section('localhost')
    cfg.set('localhost', 'ip' ,'127.0.0.1')
    cfg.set('localhost', 'port' ,'3306')
    cfg.set('localhost', 'user' ,'root')
    cfg.set('localhost', 'password' ,'root')

    cfg.add_section('interval')
    cfg.set('interval', 'heartbeat', '1')
    cfg.set('interval', 'upload_data', '5')

    cfg.write(open(os.getcwd() + '/sql.conf', 'w+'))

# 文件存在就打印出来
else:
    cfg = ConfigParser.ConfigParser()
    cfg.read(os.getcwd() + '/sql.conf')
    print cfg.items('remote')    # 获取remote中的所有内容，返回字典。
    print cfg.get('remote', 'ip')    # 获取remote中,ip的值
    print cfg.getint('remote', 'port')    # 获取remote中,port的值，并转为int型
    print cfg.get('remote', 'user')
    print cfg.get('remote', 'password')
    print cfg.get('remote', 'table')

    print cfg.items('localhost')
    print cfg.get('remote', 'ip')        
    print cfg.getint('remote', 'port')
    print cfg.get('remote', 'user')
    print cfg.get('remote', 'password')

    print cfg.items('interval')
    print cfg.get('interval', 'heartbeat')
    print cfg.get('interval', 'upload_data')
```

第一次运行创建文件

```
[remote]
ip = 192.168.3.123
port = 40949
user = tony
password = tony
table = real_tabel

[localhost]
ip = 127.0.0.1
port = 3306
user = root
password = root

[interval]
heartbeat = 1
upload_data = 5
```

第二次输出结果。

```
[('ip', '192.168.3.123'), ('port', '40949'), ('user', 'tony'), ('password', 'tony'), ('table', 'real_tabel')]
192.168.3.123
40949
tony
tony
real_tabel
[('ip', '127.0.0.1'), ('port', '3306'), ('user', 'root'), ('password', 'root')]
192.168.3.123
40949
tony
tony
[('heartbeat', '1'), ('upload_data', '5')]
1
5
```

Tony Liu

2017-6-9, Shenzhen