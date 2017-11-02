## Android绘图基础——练习一

### 仿华为加载动画

> 一直觉得华为的加载小动画挺好的，既然前面把Path的相关知识复习了一下。那我们就做这个当做我们绘图基础的一个小练习。

#### 一、练习点
- path的基础使用
- 画笔的基础使用
- pathMeasure的基础使用
- ValueAnimator的基础使用
- Interpolator插值器的使用

#### 二、思路

我们前面可以看到，这个华为的加载圆圈的速度是先快后慢。所以此处我们使用Interolator中的AccelerateDecelerateInterpolator来满足当前位置的变换。然后通过PathMeasure获取当前运行的location坐标，最后绘制出来。

#### 三、代码编写

```java
/**
 * Created by Iflytek_dsw on 2017/6/20.
 */
public class HWLoadingView extends View {
    private PathMeasure pathMeasure;
    /**当前绘制圆弧的路径*/
    private Path pathCircle;
    /**测量路径圆的半径*/
    private Path pathMeasureCircle;
    /** 绘制圆圈的坐标x、y */
    private float x,y;
    /**圆圈的半径*/
    private static final int radius = 10;
    private static int strokeWidth = 4;
    /**圆圈的颜色*/
    private int circleColor = Color.parseColor("#28C0C6");
    /**圆弧的颜色*/
    private int circlePathColor = Color.parseColor("#C3C3C3");
    /**圆圈的画笔*/
    private Paint mCirclePaint;
    /**圆弧的画笔*/
    private Paint mCirclePathPaint;
    /**当前位置的坐标*/
    private float[] currentLocation = new float[2];
    private float[] currentTan = new float[2];
    public HWLoadingView(Context context, AttributeSet attrs) {
        super(context, attrs);
        mCirclePaint = new Paint();
        mCirclePaint.setColor(circleColor);
        mCirclePaint.setStyle(Paint.Style.FILL);
        mCirclePathPaint = new Paint();
        mCirclePathPaint.setStrokeWidth(strokeWidth);
        mCirclePathPaint.setColor(circlePathColor);
        mCirclePathPaint.setStyle(Paint.Style.STROKE);
    }


    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        //此处创建Path外围圆的路径
        pathCircle = new Path();
        pathMeasureCircle = new Path();
        pathCircle.addArc(new RectF(radius , radius, w/2 - radius, h/2 -radius), -90, 360);
        pathMeasureCircle.addArc(new RectF(radius , radius, w/2 - radius, h/2 -radius), -90, 359);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.drawPath(pathCircle, mCirclePathPaint);
        if(x == 0 && y == 0){
            startAnimation();
        }else{
            canvas.drawCircle(x ,y, radius, mCirclePaint);
        }
    }

    /**
     * 开始绘制变换动画
     */
    private void startAnimation(){
        /**创建一个绑定路径Path的PathMeasure对象*/
        pathMeasure = new PathMeasure(pathMeasureCircle, false);
        ValueAnimator valueAnimator = ValueAnimator.ofFloat(0, pathMeasure.getLength());
        valueAnimator.setDuration(3000);
        valueAnimator.setRepeatCount(-1);
        valueAnimator.setInterpolator(new AccelerateDecelerateInterpolator());
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                //获取当前计算的值
                float len = (float) animation.getAnimatedValue();
                pathMeasure.getPosTan(len, currentLocation,currentTan);
                x = currentLocation[0];
                y = currentLocation[1];
                invalidate();
            }
        });
        valueAnimator.start();
    }
}
```

#### 四、View的使用

1、main_layout布局中

```javascript
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:id="@+id/activity_main"
    android:layout_width="match_parent" android:layout_height="match_parent"
    android:layout_marginBottom="@dimen/activity_vertical_margin"
    android:layout_marginLeft="@dimen/activity_horizontal_margin"
    android:layout_marginRight="@dimen/activity_horizontal_margin"
    android:layout_marginTop="@dimen/activity_vertical_margin"
    tools:context="com.dsw.hwloading.MainActivity">

    <com.dsw.hwloading.HWLoadingView
        android:layout_width="200dp"
        android:layout_height="200dp" />
</RelativeLayout>
```

2、ManiActivity中

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```
![loading](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/loading.gif)


至此，完成第一个练习。

