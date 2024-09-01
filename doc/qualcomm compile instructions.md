qualcomm编译指令

### Compile the Entire Android Software

```
source build/envsetup.sh

lunch msm8909-userdebug

make -jn (-n means the thread numbers of CPU)

j表示cpu线程的个数。开启几个线程进行编译。一般和cpu的核心数相同，双核j选2,四核j选4.
```

After compiling, it will generate many BIN files in directory of `out/target/product/msm8909`

### Compiling Different Parts of Android

```
1. Compile aboot:
Input Command:
	make aboot -jn
Target Folder:
	work/LINUX/android/out/target/product/msm8909/
Target File:
	emmc_appsboot.mbn

2. Compile kernel:

Input Command:
	make bootimage -jn
Target Folder:
	work/LINUX/android/out/target/product/msm8909
Target File:
	boot.img

3. Compile system:
Input Command:
	make systemimage -jn
Target Folder:
	work/LINUX/android/out/target/product/msm8909
Target File:
	system.img

4. Compile userdata:
Input Command:
	make userdataimage -jn
Target Folder:
	work/LINUX/android/out/target/product/msm8909
Target File:
	userdata.img

5. Compile recovery:
Input Command:
	make recoveryimage -jn
Target Folder:
	work/LINUX/android/out/target/product/msm8909
Target File:
	recovery.img
```

Tony Liu

2017-4-28, Shenzhen