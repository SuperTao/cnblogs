SoundPool主要用于播放时间较短的音效，使用soundPool占用的资源也不会太大。

### 参考链接

http://o7planning.org/en/10523/playing-sound-effects-in-android-with-soundpool

http://www.cnblogs.com/plokmju/p/android_SoundPool.html

### Example

创建一个按键，用于播放声音。打开app时，连续3次播放声音，每次点击按键一次，播放一次声音。

要播放的声音资源放在res/raw/目录中。

```
package com.example.soundpooltest;

import android.media.AudioAttributes;
import android.media.AudioManager;
import android.media.SoundPool;
import android.os.Build;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    public SoundPool pool;

    Button btn;

    int soundId;

    static final int MAX_STREAMS = 10;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // SDK21以上已经有新的操作方法，原来的方法不推荐。
        // For Android SDK >= 21
        if (Build.VERSION.SDK_INT >= 21 ) {
            AudioAttributes audioAttrib = new AudioAttributes.Builder()
                    .setUsage(AudioAttributes.USAGE_GAME)
                    .setContentType(AudioAttributes.CONTENT_TYPE_SONIFICATION)
                    .build();

            SoundPool.Builder builder = new SoundPool.Builder();
            builder.setAudioAttributes(audioAttrib).setMaxStreams(MAX_STREAMS);

            this.pool = builder.build();
        }
        // for Android SDK < 21
        else {
            /*  SoundPool(int maxStreams, int streamType, int srcQuality)
             *  maxStream:soundPoll可以支持的最大声音数量，可以添加多个声音道SoundPool中
             *  streamType:声音类型
             *  srcQuality: 声音品质
             */
            this.pool = new SoundPool(MAX_STREAMS, AudioManager.STREAM_RING, 0);
        }
        /* 添加声音到SoundPool中
         * 第一个参数，上下文参数
         * 第二个参数声音的id，我将声音文件是res/raw/carina.wav
         * 第三个参数声音的优先级,当有多个声音冲突时，优先级高的先播放
         */
        soundId = pool.load(this, R.raw.carina, 1);

        pool.setOnLoadCompleteListener(new SoundPool.OnLoadCompleteListener() {
            @Override
            public void onLoadComplete(SoundPool soundPool, int sampleId, int status) {
                btn = (Button) findViewById(R.id.play);
                btn.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        /* int play(int soundID, float leftVolume, float rightVolume, int priority, int loop, float rate)
                         * soundID: 声音的ID
                         * leftVolume/rightVolume: 左右的音量
                         * priority: 声音优先级
                         * loop: 循环播放次数，0表示不循环， -1是一直循环
                         * rate: 播放的速率
                         */
                        pool.play(soundId, 1.0f, 1.0f, 0, 0, 1.0f);
                    }
                });
                // 加载成功之后循环播放3次
                pool.play(soundId, 1, 1, 0, 3, 1);
            }
        });
    }

    @Override
    protected void onStop()
    {
        super.onStop();
        pool.release();
    }
}
```

Tony Liu

2017-3-9, Shenzhen