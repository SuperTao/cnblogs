学习android的hal层，有些书上都是拿mokoid的LetTest作为范例，看了一下，写的很比较完整。所以就拿来分析一下。

参考链接：

http://blog.csdn.net/luoshengyang/article/details/6567257

mokoid源码：

https://github.com/jollen/android-framework-mokoid

#### hal代码

hardware/modules/include/mokoid/led.h

```
#include <hardware/hardware.h>

#include <fcntl.h>
#include <errno.h>

#include <cutils/log.h>
#include <cutils/atomic.h>

/*****************************************************************************/
// 硬件模块结构体
// 保存模块的信息
struct led_module_t {
   struct hw_module_t common;

   int (*init_led)(struct led_control_device_t *dev);
};

// 硬件接口结构体, 保存操作函数等
struct led_control_device_t {
   struct hw_device_t common;

   int fd;              /* file descriptor of LED device */

   /* supporting control APIs go here */
   int (*set_on)(struct led_control_device_t *dev, int32_t led);
   int (*set_off)(struct led_control_device_t *dev, int32_t led);
};

/*****************************************************************************/
// 模块id
#define LED_HARDWARE_MODULE_ID "led"

/** helper APIs */
static inline int led_control_open(const struct hw_module_t* module,
        struct led_control_device_t** device) {
		// open就是后面要介绍的led_device_open函数
    return module->methods->open(module,
            LED_HARDWARE_MODULE_ID, (struct hw_device_t**)device);
}
```

hardware/modules/led/led.c

```
#define LOG_TAG "MokoidLedStub"

#include <hardware/hardware.h>

#include <fcntl.h>
#include <errno.h>

#include <cutils/log.h>
#include <cutils/atomic.h>

#include <mokoid/led.h>

int led_device_open(const struct hw_module_t*, const char*,
        struct hw_device_t**) ;
int led_device_close(struct hw_device_t*);

int led_on(struct led_control_device_t *dev, int32_t led)
{
	int fd;

	ALOGI("LED Stub: set %d on.", led);

	fd = dev->fd;
	ioctl(fd, 0, 0);

	sleep(30);
	ALOGI("LED On: wake up...");

	return 0;
}

int led_off(struct led_control_device_t *dev, int32_t led)
{
	int fd;

	ALOGI("LED Stub: set %d off.", led);

	fd = dev->fd;
	ioctl(fd, 0, 0);

	sleep(30);
	ALOGI("LED Off: wake up...");

	return 0;
}

// 打开设备
int led_device_open(const struct hw_module_t* module, const char* name,
        struct hw_device_t** device) 
{
	struct led_control_device_t *dev;

	dev = (struct led_control_device_t *)malloc(sizeof(*dev));
	memset(dev, 0, sizeof(*dev));

	dev->common.tag =  HARDWARE_DEVICE_TAG;
	dev->common.version = 0;
	dev->common.module = module;
	//LED 设备关闭
	dev->common.close = led_device_close;
	//LED set on 
	dev->set_on = led_on;
	//LED set off
	dev->set_off = led_off;
	//将hw_device_t结构体与led_device_t关联
	*device = &dev->common;
	// 打开设备，这里的操作跟linux的应用层操作是一样的，hal将这一层进行了进一步的封装
	/* open device file */
	dev->fd = open("/dev/cdata-test", O_RDWR);

success:
	return 0;
}

int led_device_close(struct hw_device_t* device)
{
	return 0;
}
// 定义模块method的open函数
struct hw_module_methods_t led_module_methods = {
    open: led_device_open
};

/**
 * instance of led_module_t 
 */
// 实例结构体
// 相当于向系统注册了一个ID为LED_HARDWARE_MODULE_ID的stub
const struct led_module_t HAL_MODULE_INFO_SYM = {
    common: {
        tag: HARDWARE_MODULE_TAG,
        version_major: 1,
        version_minor: 0,
        id: LED_HARDWARE_MODULE_ID,		// 模块ID
        name: "Sample LED Stub",
        author: "The Mokoid Open Source Project",
        methods: &led_module_methods,	// 对应上面的led_module_methods
    },

    /* supporting APIs go here. */
};
```

#### jni代码

frameworks/base/service/jni/com_mokoid_server_LedService.cpp

```
#define ALOG_TAG "MokoidPlatform"
#include "utils/Log.h"

#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <assert.h>

#include <jni.h>
#include <mokoid/led.h>

struct led_control_device_t *sLedDevice = NULL;

static jboolean
mokoid_init(JNIEnv *env, jclass clazz)
{
    led_module_t* module;

    ALOGI("LedService JNI: mokoid_init() is invoked.");
	// 根据模块的id找对对应的module
    if (hw_get_module(LED_HARDWARE_MODULE_ID, (const hw_module_t**)&module) == 0) {
        ALOGI("LedService JNI: LED Stub found.");
		// 运行module的open函数
        if (led_control_open(&module->common, &sLedDevice) == 0) {
    	    ALOGI("LedService JNI: Got Stub operations.");
            return 0;
        }
    }

    ALOGE("LedService JNI: Get Stub operations failed.");
    return -1;
}

static jboolean mokoid_setOn(JNIEnv* env, jobject thiz, jint led)
{
    ALOGI("LedService JNI: mokoid_setOn() is invoked.");

    if (sLedDevice == NULL) {
        ALOGI("LedService JNI: sLedDevice was not fetched correctly.");
        return -1;
    } else {
        return sLedDevice->set_on(sLedDevice, led);
    }

    return 0;
}

static jboolean mokoid_setOff(JNIEnv* env, jobject thiz, jint led, jfloat x,
				jobject str)
{
    ALOGI("LedService JNI: mokoid_setOff() is invoked.");

    if (sLedDevice == NULL) {
        ALOGI("LedService JNI: sLedDevice was not fetched correctly.");
        return -1;
    } else {
        return sLedDevice->set_off(sLedDevice, led);
    }

    return 0;
}

static jboolean mokoid_setName(JNIEnv* env, jobject thiz, jstring ns)
{
    // const jchar *name = env->GetStringChars(ns, NULL);   
    const char *name = env->GetStringUTFChars(ns, NULL);

    if (sLedDevice == NULL) {
        ALOGI("LedService JNI: sLedDevice was not fetched correctly.");
        return -1;
    } else {
        ALOGI("LedService JNI: device name = %s", name);
        return sLedDevice->set_name(sLedDevice, const_cast<char *>(name)); 
    }
}
// 注册jni层的调用方法
// framework中调用的时候，用_init函数名称就mokoid_init,后面的也一样
static const JNINativeMethod gMethods[] = {
    {"_init",	  	"()Z",
			(void*)mokoid_init},
    { "_set_on",          "(I)Z",
                        (void*)mokoid_setOn },
    { "_set_off",          "(I)Z",
                        (void*)mokoid_setOff },
    { "_set_name",          "(Ljava/lang/String;)Z",
                        (void*)mokoid_setName }
};
// 注册jni方法表
int registerMethods(JNIEnv* env) {
    static const char* const kClassName =
        "com/mokoid/server/LedService";
    jclass clazz;

    /* look up the class */
    clazz = env->FindClass(kClassName);
    if (clazz == NULL) {
        ALOGE("Can't find class %s\n", kClassName);
        return -1;
    }

    /* register all the methods */
    if (env->RegisterNatives(clazz, gMethods,
            sizeof(gMethods) / sizeof(gMethods[0])) != JNI_OK)
    {
        ALOGE("Failed registering methods for %s\n", kClassName);
        return -1;
    }

    /* fill out the rest of the ID cache */
    return 0;
}
// 注册
jint JNI_OnLoad(JavaVM* vm, void* reserved) {
    JNIEnv* env = NULL;
    jint result = -1;

    if (vm->GetEnv((void**) &env, JNI_VERSION_1_4) != JNI_OK) {
        ALOGE("ERROR: GetEnv failed\n");
	goto bail;
    }
    assert(env != NULL);

    registerMethods(env);

    /* success -- return valid version number */
    result = JNI_VERSION_1_4;

bail:
    return result;
}
```

这个部分的代码，将jni的注册也放在了同一个文件中。

一般在android的源码中，在同一级目录frameworks/base/services/jni，会有一个onload.cpp文件，专门用于注册jni。可能有多个模块需要注册,都会放到onload.cpp文件中注册。

#### framework代码

* 增加aidl文件来描述通信接口

frameworks/base/core/java/mokoid/hardware/ILedService.aidl

```
package mokoid.hardware;

/*
 * ILedService.Stub 
 */
interface ILedService
{
    boolean setOn(int led);
    boolean setOff(int led);
    boolean setAllOn();
    boolean setName(String name);
}
```

* 增加SystemServer代码

frameworks/base/service/java/com/mokoid/server/LedService.java

```
package com.mokoid.server;

import android.util.Config;
import android.util.Log;
import android.content.Context;
import android.os.Binder;
import android.os.Bundle;
import android.os.RemoteException;
import android.os.IBinder;
import mokoid.hardware.ILedService;

/**
 * expose interface to clients.
 *
 *   LedService ls = new LedService();
 *   ServiceManager.addService("led", ls);
 *
 */
public final class LedService extends ILedService.Stub {    

    static {
        System.load("/system/lib/libmokoid_runtime.so");
    }
	 
    public LedService() {
        Log.i("LedService", "Go to get LED Stub...");
	_init();
    }

    /*
     * Mokoid LED native methods.
     */
    public boolean setOn(int led) {
        Log.i("MokoidPlatform", "LED On");
	return _set_on(led, 5.1);		// 调用jni注册的函数
    }

    public boolean setOff(int led) {
        Log.i("MokoidPlatform", "LED Off");
	return _set_off(led);
    }

    public boolean setName(String name) {
	return true;
    }

    public boolean setAllOn() {
		return false;
    }

    private static native boolean _init();
    private static native boolean _set_on(int led, double d);
    private static native boolean _set_off(int led);
}
```

* 当应用层新建LedService服务时，会调用_init()函数。

* _init()在jni层注册，相当于运行了jni的mokoid_init()函数

* mokoid_init()函数中调用hw_get_module(), 通过id查找模块。

并将对应的模块信息保存到module指针中。这样，在jni层，就可以通过module这个指针访问模块的数据。

找到模块之后，调用led_control_open，打开对应的模块，并将数据保存到led模块自己创建的led_control_device_t结构体变量sLedDevice中。

sLedDevice中有文件描述符，set_on,set_off函数。在jni层通过sLedDevice与hal层交互。

* 再来看看led_control_open的细节，如上面的代码所示。

函数内部调用module->methods->open(module,LED_HARDWARE_MODULE_ID, (struct hw_device_t**)device);

module->methods->open()在注册的时候就已经定义，如下：

```
const struct led_module_t HAL_MODULE_INFO_SYM = {
    common: {
        tag: HARDWARE_MODULE_TAG,
        version_major: 1,
        version_minor: 0,
        id: LED_HARDWARE_MODULE_ID,		// 模块ID
        name: "Sample LED Stub",
        author: "The Mokoid Open Source Project",
        methods: &led_module_methods,	// 对应上面的led_module_methods
    },
```
method指向led_module_methods。

module->methods->open()就相当于调用了led_device_open()函数，调用open()函数打开设备，保存模块信息等。

这样就完成了framework到hal的一整个流程。
	
---

Tony Liu

2017-3-1, Shenzhen