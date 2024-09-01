学习android的activity，之前一直没有去琢磨，今天正好了解一下activity生命周期。

参考链接：

https://developer.android.com/guide/components/activities.html#Lifecycle

http://www.javatpoint.com/android-life-cycle-of-activity

其实上面的连接已经写的很清楚了，但对自己而言，还是做个笔记整理一下。

首先查看一张经典的图片。

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170302142229563-2007969089.png)

写个demo测试。

```
public class MainActivity extends AppCompatActivity {
    String msg = "Android : ";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.d(msg, "The onCreate() event");
    }
    @Override
    protected void onStart() {
        super.onStart();
        Log.d(msg, "The onStart() event");
    }

    @Override
    protected void onResume() {
        super.onResume();
        Log.d(msg, "The onResume() event");
    }

    @Override
    protected void onPause() {
        super.onPause();
        Log.d(msg, "The onPause() event");
    }

    @Override
    protected void onStop() {
        super.onStop();
        Log.d(msg, "The onStop() event");
    }

    @Override
    protected void onRestart() {
        super.onRestart();
        Log.d(msg, "The onRestart() event");
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d(msg, "The onDestroy() event");
    }
}
```

安装到手机中，运行。

* app启动，依次运行onCreate(),onStrat(),onResume(),这三个函数在activity不同阶段运行。

onCreate()时，activity不可见。

onStart()，activity即将可见。

onResume()，activity可见，可以和用于交互。

* 按下home键，程序中止。依次运行onPause(), onStop().

onPause()，activity不可见。

onStop()，activity不在可见。

* 再次运行app，onRestart(),onStart(),onResume()依次运行。

onRestart()在activity已经停止并即将再次调用运行。

整过过程正好对应上面的图片。

```
Activity 的整个生命周期发生在 onCreate() 调用与 onDestroy() 调用之间。您的 Activity 应在 onCreate() 中执行“全局”状态设置（例如定义布局），并释放 onDestroy() 中的所有其余资源。例如，如果您的 Activity 有一个在后台运行的线程，用于从网络上下载数据，它可能会在 onCreate() 中创建该线程，然后在 onDestroy() 中停止该线程。

Activity 的可见生命周期发生在 onStart() 调用与 onStop() 调用之间。在这段时间，用户可以在屏幕上看到 Activity 并与其交互。 例如，当一个新 Activity 启动，并且此 Activity 不再可见时，系统会调用 onStop()。您可以在调用这两个方法之间保留向用户显示 Activity 所需的资源。 例如，您可以在 onStart() 中注册一个 BroadcastReceiver 以监控影响 UI 的变化，并在用户无法再看到您显示的内容时在 onStop() 中将其取消注册。在 Activity 的整个生命周期，当 Activity 在对用户可见和隐藏两种状态中交替变化时，系统可能会多次调用 onStart() 和 onStop()。

Activity 的前台生命周期发生在 onResume() 调用与 onPause() 调用之间。在这段时间，Activity 位于屏幕上的所有其他 Activity 之前，并具有用户输入焦点。 Activity 可频繁转入和转出前台 — 例如，当设备转入休眠状态或出现对话框时，系统会调用 onPause()。 由于此状态可能经常发生转变，因此这两个方法中应采用适度轻量级的代码，以避免因转变速度慢而让用户等待。
```

Tony Liu

2017-3-2, Shenzhen