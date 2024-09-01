CountDowntimer用于倒计时。

### 参考链接

https://developer.android.com/reference/android/os/CountDownTimer.html

### 分析

CountDownTimer(long millisInFuture, long countDownInterval)

第一个参数定时时间(毫秒)，过了这段时间，定时器停止；第二个参数，时间间隔，在定时期间内，没过countDownInterval将会调用onTick函数。

cancel() 取消定时器

onFinish() 定时时间到调用

onTick(long millisUntilFinished) 时间间隔到的时候调用

start() 开始定时器

### Example

```
package com.example.countdowntimertest;

import android.os.CountDownTimer;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {
    Button btn;
    CountDownTimer timer;

    private final static long INTERVAL_TIME = 1000;
    private final static long FINISH_TIME = 1000 * 10;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

		// 定时10s, 时间间隔1s
        timer = new CountDownTimer(FINISH_TIME, INTERVAL_TIME) {
            @Override
            public void onTick(long millisUntilFinished) {
				// 打印剩余的倒计时时间，还剩几秒
                Log.d("onTick", "remaining: " + millisUntilFinished / 1000);
            }
			// 定时器时间到调用
            @Override
            public void onFinish() {
                Log.d("onTick", "timer finished");
            }
        };
		// 定时器启动 
        timer.start();

        btn = (Button) findViewById(R.id.btn);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
				// 定时器关闭
                timer.cancel();
            }
        });
    }
}
```

运行输出

```
03-22 09:51:34.432 27885-27885/com.example.countdowntimertest D/onTick: remaining: 9
03-22 09:51:34.432 27885-27885/com.example.countdowntimertest D/onTick: remaining: 8
03-22 09:51:34.432 27885-27885/com.example.countdowntimertest D/onTick: remaining: 7
03-22 09:51:35.432 27885-27885/com.example.countdowntimertest D/onTick: remaining: 6
03-22 09:51:36.433 27885-27885/com.example.countdowntimertest D/onTick: remaining: 5
03-22 09:51:37.435 27885-27885/com.example.countdowntimertest D/onTick: remaining: 4
03-22 09:51:38.436 27885-27885/com.example.countdowntimertest D/onTick: remaining: 3
03-22 09:51:39.437 27885-27885/com.example.countdowntimertest D/onTick: remaining: 2
03-22 09:51:40.438 27885-27885/com.example.countdowntimertest D/onTick: remaining: 1
03-22 09:51:42.402 27885-27885/com.example.countdowntimertest D/onTick: timer finished
```

按下button键，将会中途停止定时器。

Tony Liu

2017-3-22, Shenzhen