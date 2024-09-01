android连接了4x4的物理按键，需要映射“."。

在linux驱动层注册了按键KEY_DOT，

写android的app的时候却没有对应的宏KEYCODE_DOT。只有KEYCODE_NUMPAD_DOT。

KEYCODE_NUMPAD_DOT是针对的小键盘的"."，应该对应的是LINUX上的KEY_KPDOT。

查看KeyEvent.java有如下的一段代码。

所以KEY_DOT对应android的KEYCODE_PERIOD。

```
    /** Key code constant: '.' key. */
    public static final int KEYCODE_PERIOD          = 56;
```

Tony Liu

2017-1-16, Shenzhen