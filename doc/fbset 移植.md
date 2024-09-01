手头上的文件系统的fbset有问题，所以就自己从新移植一个到开发板上。

#### 参考链接

http://blog.chinaunix.net/uid-20768928-id-5748009.html


#### 下载地址

https://launchpad.net/ubuntu/+source/fbset/2.1-27

#### 交叉编译

解压之后

make CC=arm-linux-gcc

生成的fbset上传到开发板即可。

#### 编译问题

```
1. make: bison: Command not found

sudo apt-get install bison

2. make: flex: Command not found

sudo apt-get install flex
```

#### 设置framebuffer

```
root@freescale ~$ fbset

mode "640x480-53"
        # D: 25.000 MHz, H: 27.685 kHz, V: 52.936 Hz
        geometry 640 480 640 480 32
        timings 40000 89 164 23 10 10 10
        accel false
        rgba 8/16,8/8,8/0,8/24
endmode

设置分辨率以及时序相关的设置，调试屏幕的时候可以通过fbset来设置。

fbset -g 1366 768 1366 768 16 -t 13468 104 32 18 3 32 5

```

Tony Liu

2016-12-4, Shenzhen