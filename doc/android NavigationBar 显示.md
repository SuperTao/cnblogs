android连接7inch屏时，虚拟按键会显示到右侧，变成一条黑边，并且只有back功能。

在连接10inch的时候，虚拟按键就正常，显示在屏幕的底部。有back,home,recent app三个功能。

解决方法：

```
./myandroid/frameworks/base/policy/src/com/android/internal/policy/impl/PhoneWindowManager.java

Line 1017
 
-            mNavigationBarCanMove = true;
+           mNavigationBarCanMove = false;//true;
```

使用mmm命令，重新编译模块frameworks/base/policy/中的.mk。将生成的文件

`out/target/product/sabresd_6dq/system/framework/android.policy.jar`添加到system.img中framework目录中。


Tony Liu

2017-18, Shenzhen