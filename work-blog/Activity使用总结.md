### Activity使用总结

>作为Android入门级别的组件，Activity在Android的开发中承载了太多的东西，我们的项目开发中少不了与Activity打交道。所以需要我们熟练掌握Activity的使用。

总体介绍围绕下面的几个议题进行讨论，以前也[总结过Activity的特点](http://blog.csdn.net/mr_dsw/article/details/48198387)，那时候刚学习Android总结的也过于死板，这次准备集中把Android知识进行梳理一下，所以比较有总结性。

1. Activity的使用。
1. Activity的模式。
1. Activity的通讯
1. Activity的管理。
1. Activity常见问题：
    - 从一个应用打开另一个应用。
    - 完全退出应用。
    - 动画切换

###一、Activity常见使用
在项目的开发中，Activity承载了我们的页面展示，它通过setContentView(int Id)方法绑定显示的布局，然后进行显示。Activity的基本使用。

#### 1、Activity的定义声明
（1）、继承Activity，实现Activity的生命周期方法。

    public class MainActivity extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
        }

        @Override
        protected void onStart() {
            super.onStart();
            Log.d(TAG, "onStart");
        }

        @Override
        protected void onStop() {
            super.onStop();
            Log.d(TAG, "onStop");
        }

        @Override
        protected void onResume() {
            super.onResume();
            Log.d(TAG, "onResume");
        }

        @Override
        protected void onPause() {
            super.onPause();
            Log.d(TAG, "onPause");
        }

        @Override
        protected void onDestroy() {
            super.onDestroy();
            Log.d(TAG, "onDestroy");
        }
    }
  
（2）、在Manifest.xml清单文件中注册Activity。四大组件在使用的时候都必须在清单文件中进行列举。

    <activity
        android:name=".MainActivity"
        android:label="@string/app_name"
        android:theme="@style/AppTheme.NoActionBar" >
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />

            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
    
####2、Activity的跳转
我们知道Activity承载了我们应用几乎全部的页面显示工作，那么我们如何进行页面之前的跳转呢？这里就需要使用Intent类进行页面的跳转。

显示跳转，针对我们在同一个应用中知道跳转页面的名字。

    Intent intent = new Intent(MainActivity.this, LoginActivity.class);
	startActivity(intent);
    
隐式跳转，当我们不知道一个应用的类名，只知道action、category属性时，进行使用：

    Intent loginIntent = new Intent();
    loginIntent.setAction("com.intent.action.LOGIN");
    loginIntent.addCategory("com.intent.category.LOGIN");
    startActivity(loginIntent);

注意，这里的action和category要和声明的Activity在清单文件中注册的保持一致。隐式Intent通过Android解析，将Intent映射给可以处理此Intent的Activity、IntentReceiver或Service。

Intent解析机制主要是通过查找已注册在AndroidManifest.xml中的所有IntentFilter及其中定义的Intent，最终找到匹配的Intent。

在这个解析过程中，Android是通过Intent的action、type、category这三个属性来进行判断的，判断方法如下：

1. 如果Intent指明定了action，则目标组件的IntentFilter的action列表中就必须包含有这个action，否则不能匹配；
1. 如果Intent没有提供type，系统将从data中得到数据类型。和action一样，目标组件的数据类型列表中必须包含Intent的数据类型，否则不能匹配。 
1. 如果Intent中的数据不是content: 类型的URI，而且Intent也没有明确指定它的type，将根据Intent中数据的scheme （比如 http: 或者mailto:） 进行匹配。同上，Intent 的scheme必须出现在目标组件的scheme列表中。
1. 如果Intent指定了一个或多个category，这些类别必须全部出现在组建的类别列表中。比如Intent中包含了两个类别：LAUNCHER_CATEGORY 和 ALTERNATIVE_CATEGORY，解析得到的目标组件必须至少包含这两个类别。
1. 每一个通过startActivity()方法发出的隐式Intent都至少有一个category，就是 "android.intent.category.DEFAULT"，所以只要是想接收一个隐式Intent的Activity都应括"android.intent.category.DEFAULT" category，不然将导致 Intent 匹配失败。

####3、Activity的生命周期
- onCreate():启动创建Activity，系统第一个执行的方法。
- onStart()：就在（just before）Activity成为可见之前调用
- onRestart()：Activity从后台重新回到前台时被调用，用户可见但不可交互
- onResume()：就在（just before）Activity开始与用户交互之前调用。（此时不能完全交互）
- onWindowFocusChanged()：Activity窗口获得或失去焦点时被调用,在onResume之后或onPause之后 
- onPause()：Activity被覆盖到下面或者锁屏时被调用
- onStop()：退出当前Activity或者跳转到新Activity时被调用  
- onDestroy()：退出当前Activity时被调用,调用之后Activity就结束了
- onSaveInstanceState(Bundle outState):Activity被系统杀死时被调用. 例如:屏幕方向改变时,Activity被销毁再重建;当前Activity处于后台,系统资源紧张将其杀死. 另外,当跳转到其他Activity或者按Home键回到主屏时该方法也会被调用,系统是为了保存当前View组件的状态. 
在onPause之前被调用. 
- onRestoreInstanceState(Bundle savedInstanceState)：Activity被系统杀死后再重建时被调用.例如:屏幕方向改变时,Activity被销毁再重建;当前Activity处于后台,系统资源紧张将其杀死,用户又启动该Activity.这两种情况下onRestoreInstanceState都会被调用,在onStart之后。

在Activity的生命周期中，我们经常讨论的前台生命周期（onResume——onPause）,可视声明周期，这里我们很少谈论真正在哪个阶段我们是可以与界面交互的。我们都知道在onCreate()方法中对一个View进行getHeight()、getWidth()会获取空值，为什么呢？这是因为此时的View还没有进行宽高的测量，在Activity的生命周期中，真正使我们开始于Activity交互的周期在	onWindowFocusChanged()中，我们可以获取控件的高度和宽度。站在Window视图的角度来看，Activity的生命周期又可具体到：

* entry: onStart---->onResume---->onAttachedToWindow----------->onWindowVisibilityChanged--visibility=0---------->onWindowFocusChanged(true)------->
* exit:  onPause---->onStop---->onWindowFocusChanged(false)  ---------------------- (lockscreen)
* exit : onPause----->onWindowFocusChanged(false)-------->onWindowVisibilityChanged--visibility=8------------>onStop(to another activity)

####4、Activity的基本模式
在Android开发中，Activity有四种模式，我们可以在Manifest.xml文件中注册时进行设置：

	<activity  
    android:name=".A1"  
    android:launchMode="standard|singleTop|singleTask|singleInstance" />  
    
这里，我们需要补充一点知识，关于Activity的管理方式，这里是借助栈类型数据结构进行管理，所以这四种模式分别对应了Activity的四种管理方式：

* standard：标准默认模式，每次创建一个Activity都会创建一个对应的实例。
* singleTop：可以有多个实例，如果Activity在栈顶的时候，启动相同的Activity，不会创建新的实例，而会调用其onNewIntent方法。
* singleTask：只有一个实例。在同一个应用程序中启动他的时候，若Activity不存在，则会在当前task创建一个新的实例，若存在，则会把task中在其之上的其它Activity destory掉并调用它的onNewIntent方法。
* singleInstance：只有一个实例，并且这个实例独立运行在一个task中，这个task只有这个实例，不允许有别的Activity存在。

合理运用这四种模式，能够使应用的体验和资源使用率更好。比如：singleTask适合作为程序入口点，因为app的入口点就一个。singleInstance适合需要与程序分离开的页面，例如闹铃提醒，将闹铃提醒与闹铃设置分离。

####5、Activity的通讯数据传递
在同一个进程中的Activity之间少不了交互，那么Activity之间怎么进行数据的交互呢？

- Intent承载传递数据
- StartActivityForResult
- 广播
- 第三方通讯组件

这里，我们主要介绍下前两种的使用，后两种以后再介绍。
（1）、Intent传递数据
在Android中提供了Intent机制来协助应用间的交互与通讯，Intent负责对应用中一次操作的动作、动作涉及数据、附加数据进行描述，Android则根据此Intent的描述，负责找到对应的组件，将 Intent传递给调用的组件，并完成组件的调用。Intent有六大属性值需要注意：

* Action：用于指定我们要完成的动作，是一个字符串常量。当然我们在注册四大组件时也可以指定对应的action值。系统也有些自带的：ACTION_CALL、ACTION_MAIN。通过setAction()方法进行设置。
* Data：Intent的Data属性是执行动作的URI和MIME类型，不同的Action有不同的Data数据指定。比如：ACTION_EDIT Action应该和要编辑的文档URI Data匹配，ACTION_VIEW应用应该和要显示的URI匹配。通过setData()方法进行设置。
* Category：Intent中的Category属性是一个执行动作Action的附加信息。比如：CATEGORY_HOME则表示放回到Home界面，ALTERNATIVE_CATEGORY表示当前的Intent是一系列的可选动作中的一个。通过addCategory()进行设置。
* Type：Intent的Type属性显式指定Intent的数据类型（MIME）。一般Intent的数据类型能够根据数据本身进行判定，但是通过设置这个属性，可以强制采用显式指定的类型而不再进行推导。通过setType()方法进行设置。
* Compent：Intent的Compent属性指定Intent的的目标组件的类名称。通常 Android会根据Intent 中包含的其它属性的信息，比如action、data/type、category进行查找，最终找到一个与之匹配的目标组件。但是，如果 component这个属性有指定的话，将直接使用它指定的组件，而不再执行上述查找过程。指定了这个属性以后，Intent的其它所有属性都是可选的。（很重要，比如我们在一个应用启动另一个应用）。通过setComponent(ComponentName)方法进行设置。
* Extra：Intent的Extra属性是添加一些组件的附加信息。比如，如果我们要通过一个Activity来发送一个Email，就可以通过Extra属性来添加subject和body。通过putExtra()方法进行设置。

在上面的介绍中，我们可以发现Intent超级强大的用途，可用于启动组件，也可以用于传递数据，所以Intent的设计就是为了在组件之间进行“沟通”传递使用。在启动Activity的时候，我们可以使用putExtra方法进行设置数据。然后传递到跳转到的页面。

（2）