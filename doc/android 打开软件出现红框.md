android打开软件的时候会出现红框，剑锋之前解了这个问题。fork过来，方便以后查看。

参考链接：

　　http://www.cnblogs.com/zengjfgit/p/5377744.html

##### 一、通过Settings修改

　　1. Open Settings> Developer Options and scroll down a little.

　　2. Here you would find the Strict Mode option.

　　3. Just uncheck/unmark it.

　　4. And, then reboot your device.

##### 二、通过build.prop修改

　　persist.sys.strictmode.visual=0

　　persist.sys.strictmode.disable=1

##### Author

Tony Liu

2016-8-25, Shenzhen