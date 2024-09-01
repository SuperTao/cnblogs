调试android4.2的关机功能，希望再按下power按键长时间之后就关机。不弹出选框。

#### 参考链接

http://www.cnblogs.com/sardine/archive/2011/07/26/2117510.html

http://www.cnblogs.com/sardine/archive/2011/07/26/2117510.html

http://blog.csdn.net/jinlu7611/article/details/51354632

#### imx6源码位置

frameworks/base/policy/src/com/android/internal/policy/impl/PhoneWindowManager.java

更改部分如下

```
    private final Runnable mPowerLongPress = new Runnable() {
        @Override
        public void run() {
            // The context isn't read
            if (mLongPressOnPowerBehavior < 0) { 
                mLongPressOnPowerBehavior = mContext.getResources().getInteger(
                        com.android.internal.R.integer.config_longPressOnPowerBehavior);
            }
            int resolvedBehavior = mLongPressOnPowerBehavior;
            if (FactoryTest.isLongPressOnPowerOffEnabled()) {
                resolvedBehavior = LONG_PRESS_POWER_SHUT_OFF_NO_CONFIRM;
            }

            switch (resolvedBehavior) {
            case LONG_PRESS_POWER_NOTHING:
                break;
            case LONG_PRESS_POWER_GLOBAL_ACTIONS:
                mPowerKeyHandled = true;
                if (!performHapticFeedbackLw(null, HapticFeedbackConstants.LONG_PRESS, false)) {
                    performAuditoryFeedbackForAccessibilityIfNeed();
                }
                sendCloseSystemWindows(SYSTEM_DIALOG_REASON_GLOBAL_ACTIONS);
                // 注释掉下面的代码，会弹出选框，关机，飞行模式或者静音。再根据选择进行处理。
                //showGlobalActionsDialog();
                // add by Tony, 2017-2-6
				// 添加关闭电源的功能
                mWindowManagerFuncs.shutdown(resolvedBehavior == LONG_PRESS_POWER_SHUT_OFF);
                break;
		...
```

使用mmm命令，重新编译模块frameworks/base/policy/中的.mk。将生成的文件

`out/target/product/sabresd_6dq/system/framework/android.policy.jar`添加到system.img中framework目录中。

根据硬件设计，在kernel还添加了关闭电源的功能。否则断电就会失败。

---

Tony Liu

2017-2-6, Shenzhen