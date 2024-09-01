调用python的库，学习raspberryPi的摄像头操作方法。

参考链接:

　　https://www.raspberrypi.org/learning/getting-started-with-picamera/worksheet/

由于使用的是windows的远程桌面，所以不能够实时显示摄像头的图像。只能使用python的capture函数进行捕捉图片。


代码如下：

```
from picamera import PiCamera
from time import sleep

camera = PiCamera()
camera.start_preview()
sleep(10)
camera.capture('/home/pi/Desktop/image.jpg')
camera.stop_preview()
```

Tony Liu

2016-8-27， Shenzhen