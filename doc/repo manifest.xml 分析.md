repo是用于管理android的git仓库的工具。

之前想将android的代码放在github上面，并通过repo进行管理。但一直不知道怎么添加进去，那么多的git仓库，难道都要手动建立吗?

直到看到有人在github上面部署成功，分析一下他的manifest.xml才知道是怎么部署。

#### 参考链接

http://blog.csdn.net/billpig/article/details/7604828

http://blog.csdn.net/shift_wwx/article/details/19557031

#### manifest.xml

分析的manifest.xml文件来源： 

https://github.com/CyanogenMod/android

分析其中的default.xml，如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>

  <remote  name="github"						// 远程服务器名称是“github”，后面用github表示fetch
           fetch=".."							// 获取数据的位置是".."，上一级目录
           review="review.cyanogenmod.org" />	// gerrit审核的位置	

  <remote  name="private"						// 远程服务器名称“private”
           fetch="ssh://git@github.com" />		// 从”ssh://git@github.com下载代码

  <remote  name="aosp"							// aosp
           fetch="https://android.googlesource.com"		// 代码下载地址
           review="android-review.googlesource.com"
           revision="refs/tags/android-7.1.1_r6" />		// 默认的git分支

	// 默认的下载地址，如果没有指定remote的话。
  <default revision="refs/heads/cm-14.1"		// 默认的代码下载地址
           remote="github"						// github,表示上面的remote设置的name="github"的一项，那么下载的地址fetch就是”..“
           sync-c="true"						// 只同步指定的分支
           sync-j="4" />						// repo sync 默认的并行数目

  <!-- AOSP Projects -->
// path：将代码下载到本地的build目录中
// name：${remote fetch}/${project name}.git 
// remote 没有指定，那么久采用default地址，name=github,从”.."上一层目录下载。
// 结合name的值，就从../CyanogenMod/android_build.git这个仓库下载地址。查看作者github仓库，就能找到android_build这个仓库。
  <project path="build" name="CyanogenMod/android_build" groups="pdk,tradefed">
    <copyfile src="core/root.mk" dest="Makefile" />
  </project>
  
// 将代码下载到本地的build/blueprint目录，从remote="aosp“下载，也就是https://android.googlesource.com
// 在结合project的name,下载的仓库地址就是https://android.googlesource.com/platform/build/blueprint
  <project path="build/blueprint" name="platform/build/blueprint" groups="pdk,tradefed" remote="aosp" />
  
// 从android_buld_kati.git仓库下载，放到本地的build/kati目录。
  <project path="build/kati" name="CyanogenMod/android_build_kati" groups="pdk,tradefed" />
  <project path="build/soong" name="platform/build/soong" groups="pdk,tradefed" remote="aosp" >
    <linkfile src="root.bp" dest="Android.bp" />
    <linkfile src="bootstrap.bash" dest="bootstrap.bash" />
  </project>
  <project path="abi/cpp" name="platform/abi/cpp" groups="pdk" remote="aosp" />
  <project path="art" name="CyanogenMod/android_art" groups="pdk" />
  <project path="bionic" name="CyanogenMod/android_bionic" groups="pdk" />
  <project path="bootable/recovery" name="CyanogenMod/android_bootable_recovery" groups="pdk" />
  <project path="cts" name="platform/cts" groups="cts,pdk-cw-fs,pdk-fs" remote="aosp" />
  <project path="dalvik" name="CyanogenMod/android_dalvik" groups="pdk-cw-fs,pdk-fs" />
  <project path="developers/build" name="platform/developers/build" remote="aosp" />
  <project path="development" name="CyanogenMod/android_development" groups="pdk-cw-fs,pdk-fs" />
  <project path="device/common" name="device/common" groups="pdk-cw-fs,pdk-fs" remote="aosp" />```
  ...
```

通过分析可以看出，作者将android的代码放在github上，并通过repo进行管理。

* 自己需要改动的代码就建立仓库，放在自己的github上。

* 不改动的代码就引用google原生的代码，这样就不需要在github上面建立仓库。

如果android的源码托管在github上的话可以采用这种方式，建立仓库的数量。一个android里面的git仓库接近500个。建立仓库，保存改动的代码，是个不错的方法。

当然也有很多人在自己的私有服务器搭建git仓库，放置android源码。

---

Tony Liu

2017-2-22, Shenzhen