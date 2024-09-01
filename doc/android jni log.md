在编写的jni时，经常需要输出打印信息进行调试，而C中printf在jni中没有效果，这时就需要使用NDK提供的函数。

###### 1. jni中包含头文件

```
#include <android/log.h>
```
头文件中包含的函数都可以使用

###### 2. 添加ndk对log支持

```
build.gradle 
        ndk{
                //如果要答应log就需要添加, 否者会报log函数未定义
                ldLibs "log"
                //指定生成模块名字,也就是最终的动态库名hello-jni,相应库文件名libhello-jni.so moduleName "hello-jni"
                moduleName "xxxxx"
                // 指定生成哪些处理器架构的动态库文件，如果要运行在x86架构处理器一定需要指定 abiFilters "armeabi" , "x86"
                abiFilters "armeabi" , "x86"
        }
```

##### 3. 方便使用，进行宏定义。

```
#define TAG "log"
#define LOGE(fmt, args...) __android_log_print(ANDROID_LOG_ERROR, TAG, fmt, ##args)
```

##### Author 

Tony Liu

2016-8-13, Shenzhen