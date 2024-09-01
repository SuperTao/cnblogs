使用eclipse编写android的app时，出现错误：Unable to resolve target 'android-20'。

参看链接：

http://blog.csdn.net/u013403478/article/details/21736795

http://www.dabu.info/unable-to-resolve-target-android-xx-solution.html

出现错误的原因eclipse所使用的的sdk版本和导入工程的不同。

根据链接中的方法，找到SDK的版本，我使用的SDK的api level是18。

更改project.properties,更改为如下

`target=android-18`

Tony Liu

2017-2-13, Shenzhen