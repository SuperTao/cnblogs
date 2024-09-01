触摸屏幕时，没搞懂每个事件的启动顺序。本文记录onTouchEvent发生时，每个事件启动的顺序。

##### 测试代码

---

```
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int pointCount = event.getPointerCount();
        switch (pointCount) {
            case 1:
                switch (event.getAction() & MotionEvent.ACTION_MASK) {
                    case MotionEvent.ACTION_DOWN:
                        Log.e("touch", "1 down-------");
                        break;
                    case MotionEvent.ACTION_MOVE:
                        Log.e("touch", "1 move-------");
                        break;
                    case MotionEvent.ACTION_UP:
                        Log.e("touch", "1 up-------");
                        break;
                }
            break;
            case 2:
        switch (event.getAction() & MotionEvent.ACTION_MASK) {
            case MotionEvent.ACTION_POINTER_DOWN:
                Log.e("touch", "2 down-------");
                break;
            case MotionEvent.ACTION_MOVE:
                Log.e("touch", "2 move-------");
                break;
            case MotionEvent.ACTION_POINTER_UP:
                Log.e("touch", "2 up-------");
                break;
            }
                break;
            case 3:
                switch (event.getAction() & MotionEvent.ACTION_MASK) {
                    case MotionEvent.ACTION_POINTER_DOWN:
                        Log.e("touch", "3 down-------");
                        break;
                    case MotionEvent.ACTION_MOVE:
                        Log.e("touch", "3 move-------");
                        break;
                    case MotionEvent.ACTION_POINTER_UP:
                        Log.e("touch", "3 up-------");
                        break;
                }
                break;
        }
        return true;
```

##### 输出结果

---

单点触摸
```
07-17 11:29:38.651 15874-15874/com.example.tony.ring E/touch: 1 down-------
07-17 11:29:38.664 15874-15874/com.example.tony.ring E/touch: 1 move-------
07-17 11:29:38.677 15874-15874/com.example.tony.ring E/touch: 1 up-------
```

两点触摸
```
07-17 11:30:36.610 15874-15874/com.example.tony.ring E/touch: 1 down-------
07-17 11:30:36.611 15874-15874/com.example.tony.ring E/touch: 1 move-------
07-17 11:30:36.612 15874-15874/com.example.tony.ring E/touch: 2 down-------
07-17 11:30:36.637 15874-15874/com.example.tony.ring E/touch: 2 move-------
07-17 11:30:36.637 15874-15874/com.example.tony.ring E/touch: 2 up-------
07-17 11:30:36.638 15874-15874/com.example.tony.ring E/touch: 1 up-------
或者
07-17 11:31:43.909 15874-15874/com.example.tony.ring E/touch: 1 down-------
07-17 11:31:43.910 15874-15874/com.example.tony.ring E/touch: 2 down-------
07-17 11:31:43.942 15874-15874/com.example.tony.ring E/touch: 2 move-------
07-17 11:31:43.943 15874-15874/com.example.tony.ring E/touch: 2 up-------
07-17 11:31:43.943 15874-15874/com.example.tony.ring E/touch: 1 up-------
```

三点触摸
```
07-17 11:32:46.598 15874-15874/com.example.tony.ring E/touch: 1 down-------
07-17 11:32:46.602 15874-15874/com.example.tony.ring E/touch: 2 down-------
07-17 11:32:46.604 15874-15874/com.example.tony.ring E/touch: 3 down-------
07-17 11:32:46.604 15874-15874/com.example.tony.ring E/touch: 3 move-------
07-17 11:32:46.604 15874-15874/com.example.tony.ring E/touch: 3 up-------
07-17 11:32:46.605 15874-15874/com.example.tony.ring E/touch: 2 move-------
07-17 11:32:46.605 15874-15874/com.example.tony.ring E/touch: 2 up-------
07-17 11:32:46.607 15874-15874/com.example.tony.ring E/touch: 1 up-------
或者
07-17 11:33:06.252 15874-15874/com.example.tony.ring E/touch: 1 down-------
07-17 11:33:06.254 15874-15874/com.example.tony.ring E/touch: 2 down-------
07-17 11:33:06.256 15874-15874/com.example.tony.ring E/touch: 3 down-------
07-17 11:33:06.271 15874-15874/com.example.tony.ring E/touch: 3 move-------
07-17 11:33:06.288 15874-15874/com.example.tony.ring E/touch: 3 move-------
07-17 11:33:06.288 15874-15874/com.example.tony.ring E/touch: 3 up-------
07-17 11:33:06.289 15874-15874/com.example.tony.ring E/touch: 2 up-------
07-17 11:33:06.289 15874-15874/com.example.tony.ring E/touch: 1 up-------
```

多点触摸时，有没有ACTION_MOVE的动作不能保证，

点击时肯定先是 

　　1 down  ->  2 down -> 3 down 

释放时

　　3 up ->  2 up -> 1 up 

##### Author

Tony Liu

2016-7-17， Shenzhen