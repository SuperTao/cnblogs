在使用bootimg tools压缩和打包android的ramdisk时，用到了cpio命令。所以了解一下用法。

#### 参考

http://man.linuxde.net/cpio

http://cn.linux.vbird.org/linux_basic/0240tarcompress.php


cpio用来建立或还原备份。

* cpio还原

cpio -i

以下是bootimg tools中unpack_ramdisk脚本

```
#!/bin/bash
# Extracts a ramdisk.cpio.gz to a ramdisk subdirectory
# Optionally taking an output to repack the ramdisk into.
# Created by CNexus - XDA Developers.

if [ -z $1 ]
then
echo "Usage: unpack_ramdisk <ramdiskFile>"
exit 0
fi

mkdir ramdisk
cd ramdisk
# gunzip解压，并用cpio还原档案
gunzip -c ../$1 | cpio -i
```

* cpio备份

cpio一般结合find一起使用

以下是bootimg tools中pack_ramdisk脚本

```
#!/bin/bash
# Repacks a ramdisk for use inside a boot.img
# Optionally taking an output to repack the ramdisk into.
# Found online, modified by CNexus - XDA Developers.

if [ -z $1 ]
then
echo "Usage: repack_ramdisk <ramdiskDirectory> [outputFile]"
exit 0
fi

cd $1
# 查找当前目录的文件，进行备份，格式是newc，在使用gzip压缩
find . | cpio -o -H newc | gzip > ../new-ramdisk.cpio.gz

if [ -z $2 ]
then
exit 0
else
mv ../new-ramdisk.cpio.gz ../$2
```

Tony Liu

2017-2-27, Shenzhen