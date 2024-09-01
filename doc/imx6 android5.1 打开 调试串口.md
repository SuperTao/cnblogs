```
imx6的工板烧录android 5.1的镜像,uboot中能使用debug口，kernel,文件系统中不能使用debug口。

打开kenel和文件系统debug口方法,在uboot的bootargs添加：

        androidboot.selinux 

Android User's Guide,中的说明。

Argument to disable selinux check and enable serial input when connecting a host computer to the target board’s USB UART port.
For details about selinux, see http://source.android.com/devices/tech/security/selinux/.

As Android Lollipop 5.1 CTS requirement: serial input should be disabled by default.
Setting this argument enables console
serial input, which violates the CTS
requirement.
```
Tony Liu

2016-11-9, Shenzhen