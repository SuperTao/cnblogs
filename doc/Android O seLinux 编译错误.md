编译android O源码，遇到错误

```
FAILED: out/target/product/hon450/obj/ETC/sepolicy_intermediates/sepolicy 
/bin/bash -c "(out/host/linux-x86/bin/secilc -M true -G -c 30 out/target/product/hon450/obj/ETC/plat_sepolicy.cil_intermediates/plat_sepolicy.cil out/target/product/hon450/obj/ETC/27.0.cil_intermediates/27.0.cil out/target/product/hon450/obj/ETC/nonplat_sepolicy.cil_intermediates/nonplat_sepolicy.cil -o out/target/product/hon450/obj/ETC/sepolicy_intermediates/sepolicy.tmp -f /dev/null ) && (out/host/linux-x86/bin/sepolicy-analyze out/target/product/hon450/obj/ETC/sepolicy_intermediates/sepolicy.tmp permissive > out/target/product/hon450/obj/ETC/sepolicy_intermediates/sepolicy.permissivedomains ) && (if [ \"userdebug\" = \"user\" -a -s out/target/product/hon450/obj/ETC/sepolicy_intermediates/sepolicy.permissivedomains ]; then               echo \"==========\" 1>&2;               echo \"ERROR: permissive domains not allowed in user builds\" 1>&2;              echo \"List of invalid domains:\" 1>&2;                 cat out/target/product/hon450/obj/ETC/sepolicy_intermediates/sepolicy.permissivedomains 1>&2;            exit 1;                 fi ) && (mv out/target/product/hon450/obj/ETC/sepolicy_intermediates/sepolicy.tmp out/target/product/hon450/obj/ETC/sepolicy_intermediates/sepolicy )"
neverallow check failed at out/target/product/hon450/obj/ETC/nonplat_sepolicy.cil_intermediates/nonplat_sepolicy.cil:4181
  (neverallow base_typeattr_49_27_0 netd_27_0 (unix_stream_socket (connectto)))
    <root>
    allow at out/target/product/hon450/obj/ETC/nonplat_sepolicy.cil_intermediates/nonplat_sepolicy.cil:7695
      (allow WifiLogger_app netd_27_0 (unix_stream_socket (connectto)))

neverallow check failed at out/target/product/hon450/obj/ETC/plat_sepolicy.cil_intermediates/plat_sepolicy.cil:4696 from system/sepolicy/public/domain.te:586
  (neverallow base_typeattr_49 netd (unix_stream_socket (connectto)))
    <root>
    allow at out/target/product/hon450/obj/ETC/nonplat_sepolicy.cil_intermediates/nonplat_sepolicy.cil:7695
      (allow WifiLogger_app netd_27_0 (unix_stream_socket (connectto)))

Failed to generate binary 
```

由于我自己添加了te文件中内容与系统冲突。如下所示：

`allow WifiLogger_app netd:unix_stream_socket connectto;`


解决办法，删除上面的语句，添加

```
net_domain(WifiLogger_app)	
unix_socket_connect(WifiLogger_app, netd, netd);
```

Tony Liu

2018-3-12