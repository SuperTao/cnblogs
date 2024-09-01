android 编译模块

在写完.c文件之后，需要加载到android上进行测试。使用arm-linux-gcc编译，并添加到android开发板上运行失败。

由于android与linux不同，需要编译成模块加载才能够正常运行。本文记录编译成模块的方法。

##### 参考链接：

 　　https://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/

##### 1. Android.mk

每一个模块都有一个Android.mk文件,将源码和Android.mk都放在一个目录中。

```
LOCAL_PATH := $(call my-dir)        # 设置当前模块的编译路径为当前文件夹路径。

include $(CLEAR_VARS)               # 清理（可能由其他模块设置过的）编译环境中用到的变量。

LOCAL_MODULE    := gpio             # 当前模块名称,编译之后生成的模块名称
LOCAL_SRC_FILES := gpio.c           # 当前模块包含的所有源代码文件
LOCAL_LDLIBS    := -llog            # 编译本模块需要使用的库

include $(BUILD_EXECUTABLE)         # 编译成可执行文件 
#include $(BUILD_SHARED_LIBRARY)    # 编译成共享库
```

##### 编译

```
source build/envsetup.sh    # 初始化编译环境，并引入一些辅助的 Shell 函数，这其中就包括后面使用的使用lunch,mmm函数。

lunch sabresd_6dq-eng       # lunch函数的参数用来指定此次编译的目标设备以及编译类型, 这里指定设备是的是imx6

mmm dir                     # 编译指定目录下的模块,这个dir目录包含上面提到的Android.mk以及源码，自行更改目录的路径
```

生成的镜像位于

　　out/target/product/sabresd_6dq/system/bin/gpio

##### Author 

Tony Liu

2016-8-13, Shenzhen