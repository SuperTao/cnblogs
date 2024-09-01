android连接了一个4x4的矩阵键盘，linux内核中注册了按键，在app中监听键盘事件。

```
package com.example.tony.keydemo;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.KeyEvent;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event)
    {
        switch (keyCode) {
            // ESC
            case KeyEvent.KEYCODE_ESCAPE:
                Log.e("KEY", "KEY_ESCAPE");
                Toast.makeText(MainActivity.this, "KEY_ESCAPE", Toast.LENGTH_SHORT).show();
                break;
            // "0"
            case KeyEvent.KEYCODE_0:
                Log.e("KEY", "KEY_0");
                Toast.makeText(MainActivity.this, "KEY_0", Toast.LENGTH_SHORT).show();
                break;
            // "1"
            case KeyEvent.KEYCODE_1:
                Log.e("KEY", "KEY_1");
                Toast.makeText(MainActivity.this, "KEY_1", Toast.LENGTH_SHORT).show();
                break;
            // OK
            case KeyEvent.KEYCODE_ENTER:
                Log.e("KEY", "KEY_ENTER");
                Toast.makeText(MainActivity.this, "KEY_ENTER", Toast.LENGTH_SHORT).show();
                break;
            // "2"
            case KeyEvent.KEYCODE_2:
                Log.e("KEY", "KEY_2");
                Toast.makeText(MainActivity.this, "KEY_2", Toast.LENGTH_SHORT).show();
                break;
            // "3"
            case KeyEvent.KEYCODE_3:
                Log.e("KEY", "KEY_3");
                Toast.makeText(MainActivity.this, "KEY_3", Toast.LENGTH_SHORT).show();
                break;
            // "4"
            case KeyEvent.KEYCODE_4:
                Log.e("KEY", "KEY_4");
                Toast.makeText(MainActivity.this, "KEY_4", Toast.LENGTH_SHORT).show();
                break;
            // "5"
            case KeyEvent.KEYCODE_5:
                Log.e("KEY", "KEY_5");
                Toast.makeText(MainActivity.this, "KEY_5", Toast.LENGTH_SHORT).show();
                break;
            // "6"
            case KeyEvent.KEYCODE_6:
                Log.e("KEY", "KEY_6");
                Toast.makeText(MainActivity.this, "KEY_6", Toast.LENGTH_SHORT).show();
                break;
            // "7'
            case KeyEvent.KEYCODE_7:
                Log.e("KEY", "KEY_7");
                Toast.makeText(MainActivity.this, "KEY_7", Toast.LENGTH_SHORT).show();
                break;
            // "8"
            case KeyEvent.KEYCODE_8:
                Log.e("KEY", "KEY_8");
                Toast.makeText(MainActivity.this, "KEY_8", Toast.LENGTH_SHORT).show();
                break;
            // "9"
            case KeyEvent.KEYCODE_9:
                Log.e("KEY", "KEY_9");
                Toast.makeText(MainActivity.this, "KEY_9", Toast.LENGTH_SHORT).show();
                break;
            // F1
            case KeyEvent.KEYCODE_F1:
                Log.e("KEY", "KEY_F1");
                Toast.makeText(MainActivity.this, "KEY_F1", Toast.LENGTH_SHORT).show();
                break;
            // "."
            case KeyEvent.KEYCODE_PERIOD:
                Log.e("KEY", "KEY_PERIOD");
                Toast.makeText(MainActivity.this, "KEY_PERIOD", Toast.LENGTH_SHORT).show();
                break;
            // F2
            case KeyEvent.KEYCODE_F2:
                Log.e("KEY", "KEY_F2");
                Toast.makeText(MainActivity.this, "KEY_F2", Toast.LENGTH_SHORT).show();
                break;
            // F3
            case KeyEvent.KEYCODE_F3:
                Log.e("KEY", "KEY_F3");
                Toast.makeText(MainActivity.this, "KEY_F3", Toast.LENGTH_SHORT).show();
                break;
        }
        return super.onKeyDown(keyCode, event);
    }
	    @Override
    public boolean onKeyUp(int keyCode, KeyEvent event)
    {
        switch (keyCode) {
		......
		}
		return super.onKeyLongPress(keyCode, event);
	}
	
}                
```
Tony Liu

2017-1-16, Shenzhen