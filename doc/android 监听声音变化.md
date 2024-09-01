新的项目需要监听android声音的变化，再做出对应的操作，从网上找了个demo验证。记录于此。

#### 参考链接

https://my.oschina.net/yuanxulong/blog/372268

```
package com.example.volumetest;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.media.AudioManager;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    AudioManager audioManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        audioManager = (AudioManager) this.getSystemService(Context.AUDIO_SERVICE);

        myRegisterReceiver();

    }

    private void myRegisterReceiver() {
        MyVolumeReceiver mVolumeReceiver = new MyVolumeReceiver();
        IntentFilter filter = new IntentFilter();
        filter.addAction("android.media.VOLUME_CHANGED_ACTION");
        registerReceiver(mVolumeReceiver, filter);
    }

    private class MyVolumeReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            if (intent.getAction().equals("android.media.VOLUME_CHANGED_ACTION")) {
                int currVolume = audioManager.getStreamVolume(AudioManager.STREAM_SYSTEM);
                //int currVolume = audioManager.getStreamVolume(AudioManager.STREAM_MUSIC);
                Toast.makeText(getApplicationContext(), currVolume + "", Toast.LENGTH_SHORT).show();
            }
        }
    }
}
```

一开始通过按键调整的是系统声音STREAM_SYSTEM，读取的是STREAM_MUSIC。一直读取不到数据，更改之后就可以了。

```
STREAM_ALARM 警报   
STREAM_MUSIC 音乐回放即媒体音量   
STREAM_NOTIFICATION 窗口顶部状态栏Notification,   
STREAM_RING 铃声   
STREAM_SYSTEM 系统   
STREAM_VOICE_CALL 通话
STREAM_DTMF 双音多频,不是很明白什么东西
```

Tony Liu

2017-5-9, Shenzhen