## Android 换肤小结
>现在很多App应用都有切换主题的功能，极大的改善了在用户体验。比如我们常见的白天/黑夜模式切换，很好的满足了在黑夜模式的体验，所以这篇文章就来总结下常见的换肤实现。

随着Android的不断发展，现在在很多的应用中都有切换主题的功能，极大了提高了app的用户体验，所以趁着这段时间工作的事情比较少，来总结下常见的android主题切换的实现方式。核心本质就是涉及到的资源或者主题所存放的路径在在哪里。主要有以下四种实现方式。

1. 将主题包（图片与配置）存到SD卡上（可通过下载或手动放入指定目录），在代码里强制从本地文件创建图片与配置文字大小、颜色等信息。
1. Android平台独有的主题设置功能，在values文件夹中定义若干种style，在Activity的onCreate中使用setTheme方法设置主题。
1. 将主题包做成APK的形式，使用远程Context的方式访问主题包中的资源。
1. 深度主题，修改framework中Resources类获取资源的流程，将资源重定向到主题包中。

方式1中：
主要涉及到资源文件的读取，通过网络下载将主题文件存放在sd卡中或者存放在其他位置，然后进行读取。
方式2：
方式2中把所有的主题都设置在style文件中，所以局限性很大，造成资源包很大，不适合版本控制，每次切换都需要重新OnCreate Activity。优点是读取速度快，使用简单方便。
方式3：
这种方式皮肤就是一个apk，apk可以随时下载，随时安装，可以控制版本动态的更新，灵活方便，因为安装在手机的内存中，不收内存卡下载的影响。同时可以独立于主程序，所以能减少主程序的体积大小。
缺点：第一次使用，需要用户安装；如果用户下载了无法使用。
方式4：
只是听说，暂时未研究。

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

![theme](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E5%88%87%E6%8D%A2%E4%B8%BB%E9%A2%98%E4%B8%8E%E6%8D%A2%E8%82%A4/theme.gif)

### 二、插件式主题
在插件式的主题方式中，需要两个apk，一个主题apk和一个主体apk，主题apk主要用于存放我们的主题相关的资源。在研究这种方式前，我也看到了很多文章介绍的需要将皮肤apk和主体apk保持相同的sharedUserId，我也试验了，这里该项也不是必要的。这里要有点需要注意：如果将皮肤apk和主体apk保持相同的sharedUserId，必须保证皮肤apk只有资源，不能在Manifest.xml中设置Activity的启动项，不然两个应用不能同时安装。不使用sharedUserId不需要注意该项。同时这里包含两种处理方式：一种是将皮肤apk和主体apk分开进行安装；另一种是将皮肤apk放到主体apk的assets文件夹中，这样在主体apk中通过AssetManager进行资源的方式，本质也是使用下面的两个方法进行资源的访问。

在该方式中主要涉及到两个方法：
1、android.content.ContextWrapper.createPackageContext(String packageName, int flags)，通过包名就可以获取一个应用程序的Context对象，然后在通过getResource方法获取皮肤apk的Resource对象。
2、getIdentifier(String name, String defType, String defPackage)，通过资源名称、类型、包名获取指定的资源id，然后通过id获取对应的资源文件。

这里我们简单演示下：
在皮肤包的资源文件中，放置一段字符串资源：

	<string name="plugin_string">plugin_string</string>
    
然后我们在主体包中获取该资源，此处为了方便，仅仅展示原理。

	public class SecondActivity extends Activity {
        private Context pluginContext;
        Resources pluginResources;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.second_layout);
            Button btn_plugin = (Button) findViewById(R.id.btn_plugin);
            try {
                pluginContext = createPackageContext("com.dsw.zingtest", Context.CONTEXT_IGNORE_SECURITY);
            } catch (NameNotFoundException e) {
                e.printStackTrace();
            }
            pluginResources = pluginContext.getResources();
            btn_plugin.setOnClickListener(new OnClickListener() {

                @Override
                public void onClick(View v) {
                    String text = pluginResources.getString(pluginResources.getIdentifier(
                            "plugin_string", "string", "com.dsw.zingtest"));
                    Toast.makeText(getApplicationContext(), text,0).show();
                }
            });
        }
    }
    
仅仅通过这两个方法就可以获取到皮肤资源的apk，然后完成安装。这个唯一的担心是客户是否愿意安装皮肤apk。

![apktheme](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E5%88%87%E6%8D%A2%E4%B8%BB%E9%A2%98%E4%B8%8E%E6%8D%A2%E8%82%A4/apktheme.gif)

同时这里提供了两个反射获取资源的工具类，供大家参考。该文章对应的github路径下可查看。

[Android切换主题工具类](https://github.com/dengshiwei/work-summary/tree/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E5%88%87%E6%8D%A2%E4%B8%BB%E9%A2%98%E4%B8%8E%E6%8D%A2%E8%82%A4)

上面的两个常用方法介绍完了，这篇文章也算完了，主要是第四中方法，估计很有难度，还没有研究，以后有缘分在研究吧！后面可能会更新一篇关于换肤的框架介绍。