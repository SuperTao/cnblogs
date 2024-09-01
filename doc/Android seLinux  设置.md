在android O上添加服务。在访问一些路径的时出现了权限的问题，将seLinux关闭之后运行成功。所以需要设置相关的权限。

参考文档：

http://blog.csdn.net/tung214/article/details/72734086

主要查看dmsg中关于avc的错误，根据对应的错误在对应的文件中添加权限。

关于权限的文件位于device/xxxx/sepolicy/common目录中。

例如init.te

```
allow init oemfs:dir { mounton };

#honeywell license check
allow init counter_sysfs:file rw_file_perms;

allow init fuse:dir { search mounton };
allow init self:capability sys_module;
allow init {
    adsprpcd_file
    cache_file
    persist_file
    storage_file
}:dir mounton;
#Allow init to mount non-hlos partitions in A/B builds
allow init media_rw_data_file:dir { mounton };
allow init unlabeled:dir { mounton };
allow init oemfs:dir { mounton };
allow sdcardd sdcardfs:dir { mounton };
```

log里面如果出现`Service xxx does not have a SELinux domain defined`,还需要添加domain，否则也是编译不过的。添加方法

http://blog.csdn.net/jianchi88/article/details/78417202


Tony Liu

2018-3-6