在jni中希望将字符串转成浮点型数据，使用了atof函数。出现错误:

`failed: Cannot load library: soinfo_relocate(linker.cpp:975): cannot locate symbol "atof" referenced by "libxxx-jni.so"...`

#### 参考

http://stackoverflow.com/questions/14571399/android-ndk-cant-find-atof-function

#### 解决方法

```
From stdlib.h in the Android source;

static __inline__ double atof(const char *nptr)
{
    return (strtod(nptr, NULL));
}

直接调用strtod(nptr,NULL)用来代替atof(nptr)函数。
```

Tony Liu

2017-1-16, Shenzhen