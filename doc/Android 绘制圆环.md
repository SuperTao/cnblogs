使用画圆弧的方式绘制圆环和进度条，使用sweepGradient进行渐变。

###### 参考链接

http://blog.csdn.net/u011494050/article/details/39251239

http://www.jianshu.com/p/6c5e4dce11aa

###### 代码

```
package com.example.tony.ring;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Matrix;
import android.graphics.Paint;
import android.graphics.RectF;
import android.graphics.Shader;
import android.graphics.SweepGradient;
import android.util.Log;
import android.view.View;

/**
 * Created by Tony on 2016/7/6.
 */
public class DrawRing extends View {
    private int ballX = 300;
    private int ballY = 300;
    private int radius = 100;
    private int width = 30;
    private int startAngle = 360;
    private int angleSpeed = 1;

    public DrawRing(Context context) {
        super(context);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        CircleRing cRing = new CircleRing(ballX, ballY, radius, width, Color.RED);
        cRing.drawCircleRing(canvas, startAngle);

        ProgressRing pRing = new ProgressRing(ballX , ballY + 300, radius, width, Color.RED);
        pRing.drawProgressRing(canvas, startAngle);

        /*
        if (startAngle <= 0) {
            angleSpeed = 1;
        } else if (startAngle >= 90) {
            angleSpeed = -1;
        }
        */
        startAngle += angleSpeed;
        if (startAngle == 360)
            startAngle = 0;
//        startAngle %= 360;
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        invalidate();
    }
}

class CircleRing {
    int ringX;
    int ringY;
    int ringRadius;
    int ringWidth;
    Paint paint;

//    public CircleRing(int ringX, int ringY, int ringRadius, int ringWidth, int ringColor, int startAngle) {
      public CircleRing(int ringX, int ringY, int ringRadius, int ringWidth, int ringColor) {
        this.ringX = ringX;
        this.ringY =  ringY;
        this.ringRadius =  ringRadius;
        this.ringWidth=  ringWidth;
        paint = new Paint();
        paint.reset();
        paint.setColor(ringColor);
        paint.setAntiAlias(true); //消除锯齿
        paint.setStrokeWidth(ringWidth);
        paint.setStyle(Paint.Style.STROKE);  //绘制空心圆或 空心矩形,只显示边缘的线，不显示内部
    }

    public void drawCircleRing(Canvas canvas, int startAngle) {
        RectF rect = new RectF(ringX - ringRadius, ringY - ringRadius, ringX + ringRadius, ringY + ringRadius);
        //false 不画圆心
        paint.setAlpha(255);
        canvas.drawArc(rect, startAngle + 0, 90, false, paint);
        canvas.drawArc(rect, startAngle + 180, 90, false, paint);
        paint.setAlpha(100);
        canvas.drawArc(rect, startAngle + 90, 90, false, paint);
        canvas.drawArc(rect, startAngle + 270, 90, false, paint);
    }
}

class ProgressRing {
    int ringX;
    int ringY;
    int ringRadius;
    int ringWidth;
    int headBallX;
    int headBallY;
    Paint paint;
    Shader  sweepGradient;
    //private static final int RED = 230, GREEN = 85, BLUE = 35; //基础颜色，这里是橙红色
    private static final int RED = 230, GREEN = 0, BLUE = 0; //基础颜色，这里是橙红色
    private static final int MIN_ALPHA = 30; //最小不透明度
    private static final int MAX_ALPHA = 255; //最大不透明度
    /*
    //圆环颜色
    //渐变顺序不同，显示的方向不同
    private static int[] changeColors = new int[]{
            Color.argb(MAX_ALPHA, RED, GREEN, BLUE),
            Color.argb(MIN_ALPHA, RED, GREEN, BLUE),
            Color.argb(MIN_ALPHA, RED, GREEN, BLUE),
//            Color.TRANSPARENT,
    };
    */
    private static int[] changeColors = new int[]{
            Color.argb(MIN_ALPHA, RED, GREEN, BLUE),
            Color.argb(MIN_ALPHA, RED, GREEN, BLUE),
            Color.argb(MAX_ALPHA, RED, GREEN, BLUE),
//            Color.TRANSPARENT,
    };

    public ProgressRing(int ringX, int ringY, int ringRadius, int ringWidth, int ringColor) {
        this.ringX = ringX;
        this.ringY =  ringY;
        this.ringRadius =  ringRadius;
        this.ringWidth=  ringWidth;
        paint = new Paint();
        paint.reset();
        paint.setColor(ringColor);
        paint.setAntiAlias(true); //消除锯齿
        paint.setStrokeWidth(ringWidth);
        paint.setStyle(Paint.Style.STROKE);  //绘制空心圆或 空心矩形,只显示边缘的线，不显示内部
    }

    public void drawProgressRing(Canvas canvas, int rotateDegree) {
        paint.reset();
        RectF rect = new RectF(ringX - ringRadius, ringY - ringRadius, ringX + ringRadius, ringY + ringRadius);
        paint.setAntiAlias(true);
        paint.setStrokeWidth(ringWidth);
        paint.setStyle(Paint.Style.STROKE);
        //设置渐变
        sweepGradient = new SweepGradient(ringX, ringY, changeColors, null);
        //按照圆心旋转
        Matrix matrix = new Matrix();
        matrix.setRotate(rotateDegree, ringX, ringY);
        sweepGradient.setLocalMatrix(matrix);
        paint.setShader(sweepGradient);
        canvas.drawArc(rect, 0, 360, false, paint);
        //绘制进度环开头的小圆点
        paint.reset();
        paint.setAntiAlias(true);
        paint.setStyle(Paint.Style.FILL);
        paint.setColor(Color.argb(MAX_ALPHA, RED, GREEN, BLUE));
        //使用三角函数时，需要把角度转为弧度
        headBallX = ringX + (int)(ringRadius * Math.cos((double)rotateDegree/180 * Math.PI));
        Log.e("degree", "degree: " + rotateDegree + "cos: " + Math.cos((double)rotateDegree/180 * Math.PI));
        headBallY = ringY + (int)(ringRadius * Math.sin((double)rotateDegree/180 * Math.PI));
        canvas.drawCircle(headBallX, headBallY, ringWidth/ 2, paint);
    }
}
```
###### 显示效果

![](http://images2015.cnblogs.com/blog/745188/201607/745188-20160706215315483-600531267.png)

###### Author

Tony Liu

2016-7-6， Shenzhen