## Android自定义Toast视图和动画

在[Android Toast基础与原理](http://blog.csdn.net/mr_dsw/article/details/74784437)中，我们对Toast的源码进行了分析。我们也对Toast的实现原理有了一定的了解。接下来我们将编写一个工具类，来完成Toast的自定义视图和动画。

### 一、实现原理分析

通过上篇文章，我们知道Toast是通过内部类TN（一个**ITransientNotification**对象）进行实现。通过与INotificationManager进行管理。在源码中，我们在TN的构造函数中：

```java
TN() {
    // XXX This should be changed to use a Dialog, with a Theme.Toast
    // defined that sets up the layout params appropriately.
    final WindowManager.LayoutParams params = mParams;
    params.height = WindowManager.LayoutParams.WRAP_CONTENT;
    params.width = WindowManager.LayoutParams.WRAP_CONTENT;
    params.format = PixelFormat.TRANSLUCENT;
    params.windowAnimations = com.android.internal.R.style.Animation_Toast;
    params.type = WindowManager.LayoutParams.TYPE_TOAST;
    params.setTitle("Toast");
    params.flags = WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON
            | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
            | WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE;
}
```

看到在params中设置Toast的显示动画。

```java
params.windowAnimations = com.android.internal.R.style.Animation_Toast;
```

所以我们的实现思路就是通过反射获取到对应的WindowManager.LayoutParams对象，然后进行设置我们的动画即可。

### 二、工具类的实现

```java
/**
 * Created by Iflytek_dsw on 2017/7/4.
 */
public class ToastManager {

    private Toast toast;
    private Context context;
    private static ToastManager instance;


    private ToastManager(Context context){
        this.context = context;
        toast = new Toast(context);
    }

    /**
     * 构造ToastManager对象
     * @param context 上下文对象
     * @return
     */
    public static ToastManager getInstance(Context context) {
        if(instance == null){
            instance = new ToastManager(context);
        }
        return instance;
    }

    /**
     * 自定义View和显示位置的Toast
     * @param view  Toast的View视图
     * @param gravity   Toast的显示位置
     * @param xOffset   Toast在x方向上偏移量,x大于0往右，x小于0往左
     * @param yOffset   Toast在y方向上偏移量,y值大于0往上，小于0往下
     */
    public void makeToastSelfView(View view, int gravity,int xOffset, int yOffset){
        if(toast == null){
            toast = new Toast(context);
        }

        toast.setView(view);
        toast.setGravity(gravity, xOffset, yOffset);
        toast.setDuration(Toast.LENGTH_SHORT);
        toast.show();
    }

    /**
     * 构造带有动画的Toast
     * @param tText 提示的文本
     * @param animationID   style封装的动画资源id
     */
    public void makeToastSelfAnimation(String tText,int animationID){
        toast = Toast.makeText(context, tText, Toast.LENGTH_SHORT);
        try {
            Field mTNField = toast.getClass().getDeclaredField("mTN");
            mTNField.setAccessible(true);
            Object mTNObject = mTNField.get(toast);
            Class tnClass = mTNObject.getClass();
            Field paramsField = tnClass.getDeclaredField("mParams");
            /**由于WindowManager.LayoutParams mParams的权限是private*/
            paramsField.setAccessible(true);
            WindowManager.LayoutParams layoutParams = (WindowManager.LayoutParams) paramsField.get(mTNObject);
            layoutParams.windowAnimations = animationID;
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        toast.show();
    }

    /**
     * 设置自定义View和Animation
     * @param view  自定义View
     * @param animationID   动画资源id
     */
    public void makeToastSelfViewAnim(View view, int animationID){
        if(toast == null){
            toast = new Toast(context);
        }

        toast.setView(view);
        try {
            Field mTNField = toast.getClass().getDeclaredField("mTN");
            mTNField.setAccessible(true);
            Object mTNObject = mTNField.get(toast);
            Class tnClass = mTNObject.getClass();
            Field paramsField = tnClass.getDeclaredField("mParams");
            /**由于WindowManager.LayoutParams mParams的权限是private*/
            paramsField.setAccessible(true);
            WindowManager.LayoutParams layoutParams = (WindowManager.LayoutParams) paramsField.get(mTNObject);
            layoutParams.windowAnimations = animationID;
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        toast.show();
    }
}
```

### 三、体验Sample

#### 1、自定义Toast的layout布局
首先我们自定义一个Toast的视图layout布局。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="wrap_content"
    android:gravity="center_vertical"
    android:background="#000000"
    android:padding="10dp"
    android:layout_height="wrap_content">
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@mipmap/dicus"/>
    <TextView
        android:id="@+id/tv_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:text="发表评论+10松果"
        android:textColor="#FFFFFF"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="20dp"
        android:text="查看"
        android:textColor="#E8C84A"/>
</LinearLayout>
```
#### 2、自定义实现动画
1、新建anim文件夹，用于存储我们的enter和out动画。

**enter**

```xml
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromXScale="0"
    android:toXScale="1"
    android:pivotX="50%"
    android:pivotY="50%"
    android:fromYScale="1"
    android:toYScale="1"
    android:duration="1000">
</scale>
```
****
**out**
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <scale
        android:fromXScale="1"
        android:toXScale="0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:fromYScale="1"
        android:toYScale="1"
        android:duration="1000"
        />
</set>
```
****
**style样式**
```xml
    <style name="MyToast" parent="Base.Animation.AppCompat.Dialog">
        <item name="android:windowEnterAnimation">@anim/enter</item>
        <item name="android:windowExitAnimation">@anim/out</item>
    </style>
```

#### 3、调用工具类实现
```java
public class MainActivity extends AppCompatActivity {
    private Button btn_toast;
    private View toastView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        LayoutInflater inflater = LayoutInflater.from(MainActivity.this);
        toastView = inflater.inflate(R.layout.toastlayout, null);
        btn_toast = (Button) findViewById(R.id.btn_toast);
        btn_toast.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                ToastManager.getInstance(getApplicationContext()).
                        makeToastSelfViewAnim(toastView,R.style.MyToast);
            }
        });
    }
}
```
效果图：

![toastview](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/toast.gif)
