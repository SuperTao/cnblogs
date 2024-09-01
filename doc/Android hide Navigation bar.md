最近一个app需要隐藏Navigation bar导航栏。

参考文档

http://blog.csdn.net/zwlove5280/article/details/52823128

http://jcodecraeer.com/a/anzhuokaifa/developer/2014/1117/1998.html

http://stackoverflow.com/questions/16713845/permanently-hide-navigation-bar-on-activity

http://stackoverflow.com/questions/27115600/android-hide-navigation-bar-stay-in-immersive-mode-with-soft-keyboard-appearance

按照参考文档添加代码。

```
			View decorView = getWindow().getDecorView();
			int options;
			options =
	                    View.SYSTEM_UI_FLAG_LAYOUT_STABLE
	        	| View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
	        	| View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
	        	| View.SYSTEM_UI_FLAG_HIDE_NAVIGATION // hide nav bar
	        	| View.SYSTEM_UI_FLAG_FULLSCREEN // hide status bar
	        	| View.SYSTEM_UI_FLAG_LOW_PROFILE
	        	| View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY;

			decorView.setSystemUiVisibility(options);
```

在onCreate()，onResume()，onWindowFocusChange(),setOnSystemUiVisibilityChangeListener都进行了尝试。

虽然可以隐藏导航栏，但是在android5.1中,当点击输入框弹出软键盘时，还是会出现导航栏。这个问题有空再来解决。

要永久删除android中导航栏。需要在/system/build.prop中添加qemu.hw.mainkeys=1

Tony Liu

2017-4-13, Shenzhen