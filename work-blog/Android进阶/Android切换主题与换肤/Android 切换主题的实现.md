## Android 实现切换主题小结
>现在很多App应用都有切换主题的功能，极大的改善了在用户体验。比如我们常见的白天/黑夜模式切换，很好的满足了在黑夜模式的体验，所以这篇文章就来总结下常见的换肤实现。

### 一、App中内置主题
这种方式通过将定义的主题文件定义在Apk中，然后完成主题的切换。通常这类不宜涉及到众多的资源，比如涉及到很多控件都需要更换背景图片资源，然后在apk中包含很多图片资源，增大apk的大小造成得不偿失。所以这类这是针对更换背景图简单的色调之类的。实现思路：

1. 定义自定义属性，并将属性值绑定到对应的控件属性上。
2. 自定义相应的Theme样式。
3. 通过setTheme给Activity设置样式。

#### 1、自定义属性，并将属性绑定到对应控件属性上。
我们在values目录中创建attrs文件，用于我们的自定义属性。

    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <attr name="TextColor" format="reference"/>
        <attr name="Backgroud" format="reference"/>
        <attr name="lineColor" format="reference"/>
        <attr name="blankColor" format="reference"/>
        <attr name="changeText" format="string"/>
        <attr name="changeDrawable" format="reference"/>
    </resources>

在上面的自定义属性中，我们定义了文本的颜色、线条的颜色等多个属性，然后将这些属性绑定到对应的控件上。比如：

    <TextView 
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        android:text="?attr/changeText"
        android:textSize="17sp"
        android:textColor="?attr/TextColor"
        android:drawableTop="?attr/changeDrawable"
        android:layout_gravity="center_horizontal"/>

绑定好属性，就开始定义Theme属性。

#### 定义Theme样式
我们创建不同的Theme，完成样式主题的验证，同时在不同的Theme样式中设置不同属性值对应的值，然后切换Theme完成主题的切换。

	<!-- 黑夜模式 -->
    <style name="NightTheme" parent="AppTheme">
        <item name="TextColor">@color/setting_text_night</item>
        <item name="Backgroud">@color/setting_back_night</item>
        <item name="lineColor">@color/setting_line_night</item>
        <item name="blankColor">@color/setting_blank_night</item>
        <item name="changeText">白天</item>
        <item name="changeDrawable">@drawable/sun</item>
    </style>
    
    <!-- 白天模式 -->
    <style name="DayTheme" parent="AppTheme">
        <item name="TextColor">@color/setting_text_day</item>
        <item name="Backgroud">@color/setting_back_day</item>
        <item name="lineColor">@color/setting_line_day</item>
        <item name="blankColor">@color/setting_blank_day</item>
        <item name="changeText">黑夜</item>
        <item name="changeDrawable">@drawable/moon</item>
    </style>

#### 在Activity中设置不同的Theme样式。
在我们进行设置的时候，我们要注意，在Android中通过setTheme完成样式的设置，但是这个方法需要在setContextView之间进行设置才有效果，所以当我们更换样式的额时候，使用此方法需要重新启动Activity。在这里我们加上一个渐变的动画，使效果体验提来不是那么突兀。

	public class MainActivity extends Activity {
        private LinearLayout linear_change;
        private static boolean useThemeBlack = false;  
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            if(getIntent() != null && getIntent().getBooleanExtra("Theme", false)){
                setTheme(R.style.NightTheme);
            }else{
                setTheme(R.style.DayTheme);
            }
            setContentView(R.layout.activity_main);
            linear_change = (LinearLayout) findViewById(R.id.linear_change);
            linear_change.setOnClickListener(new OnClickListener() {

                @Override
                public void onClick(View v) {
                    useThemeBlack = !useThemeBlack;
                    Intent intent = new Intent(MainActivity.this,MainActivity.class);
                    intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK|IntentCompat.FLAG_ACTIVITY_CLEAR_TASK);
                    intent.putExtra("Theme", useThemeBlack);
                    startActivity(intent);
                    overridePendingTransition(R.anim.slide_in, R.anim.slide_out);
                }
            });
        }
    }
    
通过一个变量来判断我们的模式切换，然后设置setTheme进行完成。同时要注意我们给启动Activity设置的Flag属性，完成一个重新启动。
