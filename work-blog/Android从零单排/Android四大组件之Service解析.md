## Android四大组件之Service解析
学习Android这么久，也没有对Service进行深入全面的学习，所以这篇文章的出发点就是全面的解析下Service。我们知道，service在我们项目中主要用于处理一些后台操作或一些长期需要处理的事务，比如常见的音乐播放，即时我们退出界面，它亦然能够为我们服务。这就是service的妙处。下面我们开始来对service进行一个全面的学习。

### 一、Service的分类
1、按照运行进程来分：

* 本地服务（Local Service）<br>
	本地服务是运行在我们应用的主进程中，与别的组件运行在同一进程中，这样做的好处是节省资源，在同一进程中，组件之间的通信方便。弊端就是如果主进程被杀死，服务便会终止。常见案例：音乐播放器
* 远程服务（Remote Service）<br>
  远程服务运行在独立的进程中，所以我们在manifest中注册该服务的时候需要android:process=":remote"来进行指定运行的进程。这样做的好处就是该服务不受其它进程的影响，便于为多个进程提供服务。弊端就是独立的进程占用一定的系统资源，通信复杂。

2、按照运行效果来分：

* 前台服务<br>
   用于由于内存不足系统杀死服务的时候能够给予用户一定的通知作用，所以前台服务必须具有一个Notificattion状态栏，即显示在通知栏上。常见的案例：手机管家、音乐播放器
* 后台服务<br>
  默认的服务，当我们启动一个服务，不做处理的时候，就是一个后台服务。常见案例：新闻的自动更新、天气的获取

### 二、Service的启动与停止
启动service由两种方式，一种是通过startService()方法借助intent来进行启动服务。例如：

	

```
Intent intent = new Intent(this, HelloService.class);
startService(intent);
```

另一种方式是通过bindService()方法来进行启动服务。例如：
	

```
Intent intent = new Intent(MainActivity.this,MyService.class);     

bindService(intent,serviceConnection,BIND_AUTO_CREATE);
```

停止Service的同样对应两种方式，当我们通过startService进行启动的时候，我们通过stopService()或stopSelf、Service.stopSelfResult()进行停止。当通过bindService方式进行启动时，我们需要调用unbindService()来停止服务。

这里就简要介绍下吧！以便我们后面测试Service的生命周期进行使用。

### 三、Service的生命周期
Android的四大组件都有生命周期，所以service也具有生命周期，供我们进行service的管理。我们先做个小案例来看看，然后再结合官网给我们提供的service生命周期图来分析下service的生命周期。

在android studio中新建一个module，命名为servicestudy。然后新建一个service的子类，MyService。
	

```
public class MyService extends Service {
	    private static final String TAG = "MyService";
	    public MyService() {
	    }
	
	    @Override
	    public IBinder onBind(Intent intent) {
	       Log.d(TAG,"onBind");
           return null;
	    }
	
	    @Override
	    public void onCreate() {
	        super.onCreate();
	        Log.d(TAG,"onCreate");
	    }
	
	    @Override
	    public void onStart(Intent intent, int startId) {
	        super.onStart(intent, startId);
	        Log.d(TAG,"onStart");
	    }
	
	    @Override
	    public int onStartCommand(Intent intent, int flags, int startId) {
	        Log.d(TAG,"onStartCommand");
	        return super.onStartCommand(intent, flags, startId);
	    }
	
	    @Override
	    public void onDestroy() {
	        Log.d(TAG,"onDestroy");
	        super.onDestroy();
	    }
	
	    @Override
	    public boolean onUnbind(Intent intent) {
	        Log.d(TAG,"onUnbind");
	        return super.onUnbind(intent);
	    }
	
	    @Override
	    public void onRebind(Intent intent) {
	        Log.d(TAG,"onRebind");
	        super.onRebind(intent);
	    }
	}
```

在MyService类中，我们没做什么业务处理，仅仅是打印Log进行测试。我们建立我们的主Activity的xml布局文件。

	

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:orientation="vertical"
    tools:context=".MainActivity">
	    <LinearLayout
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:orientation="horizontal">
	        <Button
	            android:id="@+id/btn_start"
	            android:text="start service"
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content" />
	        <Button
	            android:id="@+id/btn_stop"
	            android:text="stop service"
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content" />
	    </LinearLayout>
	    <LinearLayout
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:layout_marginTop="10dp"
	        android:orientation="horizontal">
	        <Button
	            android:id="@+id/btn_bind"
	            android:text="bind service"
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content" />
	        <Button
	            android:id="@+id/btn_unbind"
	            android:text="unbind service"
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content" />
	    </LinearLayout>
	</LinearLayout>
```

下面我们就处理四个按钮的事件，来观察Service的生命周期。

	

```
public class MainActivity extends ActionBarActivity implements View.OnClickListener {
	    private static final String TAG = "MyService:MainActivity";
	    private Button btn_startService;
	    private Button btn_stopService;
	    private Button btn_bindService;
	    private Button btn_unbindService;
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        btn_startService = (Button) findViewById(R.id.btn_start);
	        btn_stopService = (Button) findViewById(R.id.btn_stop);
	        btn_bindService = (Button) findViewById(R.id.btn_bind);
	        btn_unbindService = (Button) findViewById(R.id.btn_unbind);
	        btn_startService.setOnClickListener(this);
	        btn_stopService.setOnClickListener(this);
	        btn_bindService.setOnClickListener(this);
	        btn_unbindService.setOnClickListener(this);
	    }
	
	    @Override
	    public void onClick(View v) {
	        switch (v.getId()){
	            case R.id.btn_start:
	                Intent intentStart = new Intent(MainActivity.this,MyService.class);
	                startService(intentStart);
	                break;
	            case R.id.btn_stop:
	                Intent intentStop = new Intent(MainActivity.this,MyService.class);
	                stopService(intentStop);
	                break;
	            case R.id.btn_bind:
	                Intent intentBind = new Intent(MainActivity.this,MyService.class);
	                bindService(intentBind,serviceConnection,BIND_AUTO_CREATE);
	                break;
	            case R.id.btn_unbind:
	                unbindService(serviceConnection);
	                break;
	        }
	    }
	
	    private ServiceConnection serviceConnection = new ServiceConnection() {
	        @Override
	        public void onServiceConnected(ComponentName name, IBinder service) {
	            Log.d(TAG,"onServiceConnected");
	        }
	
	        @Override
	        public void onServiceDisconnected(ComponentName name) {
	            Log.d(TAG,"onServiceDisconnected");
	        }
	    };
	}
```

最后一步，也是很关键的一步，在Manifest中注册我们的Service：

	

```
<service
            android:name=".MyService"
            android:enabled="true"
            android:exported="true" >
    </service>
```

来看我们的实验效果：<br>
1、通过StartService()方法启动服务的生命周期：

	

```
	MyService﹕ onCreate
	MyService﹕ onStartCommand
	MyService﹕ onStart
```

我们再次点击StartService按钮，进行启动服务。看它的生命周期流程：

	

```
	MyService﹕ onStartCommand
	MyService﹕ onStart
```

通过测试可知，startService启动服务的生命周期，onCreat——>onStartCommand——>onstart，当我们启动一个已经启动的service的时候，已经不调用onCreate方法，而是直接走了onStartCommand方法。

2、通过stopService()停止服务的生命周期：

	

```
MyService﹕ onDestroy
```

直接调用service的onDestroy()方法进行service的销毁。注意使用Service的stopSelf()方法来停止服务，要注意这是一个父类的方法，调用了这个方法之后，服务停止的时间不确定，后面的代码还是会执行，并且onDestroy()方法也会执行，下次重新启动服务的时候，先调用onCreate(),然后再调用onStart()方法。

同时我们应注意到，如果同时有多个服务启动请求发送到onStartCommand(),不应该在处理完一个请求后调用stopSelf()；因为在调用此函数销毁service之前，可能service又接收到新的启动请求，如果此时service被销毁，新的请求将得不到处理。此情况应该调用stopSelf(int startId)。

当我们启动一个服务并且不需要与它进行交互的时候，我们通过startService可以很方便的进行服务的启动。

3、通过bindService进行启动服务的生命周期：

```
	MyService﹕ onCreate
	MyService﹕ onBind
```

流程是：onCreate——>onBind方法。我们再次点击bind service进行绑定，发现没有效果，我们点击unbind service进行解绑服务，看下生命周期的方法：

```
	MyService﹕ onUnbind
	MyService﹕ onDestroy
```

所以调用bindService的生命周期为：onCreate --> onBind(只一次，不可多次绑定) --> onUnbind --> onDestory。

我们知道onBind()返回给客户端一个IBind接口实例，IBind允许客户端回调服务的方法，比如得到Service的实例、运行状态或其他操作。这个时候把调用者（Context，例如Activity）会和Service绑定在一起，Context退出了，Srevice就会调用onUnbind->onDestroy相应退出。同时也实现了我们的activity等组件与service之间的通信。

4、我们点击start service然后点击bind service，看下生命周期：

```
	MyService﹕ onCreate
	MyService﹕ onStartCommand
	MyService﹕ onStart
	MyService﹕ onBind
```

然后我们点击stop service，发现系统并没有调用onDestroy方法，然后我点击unbind service方法呢？

```
	MyService﹕ onUnbind
	MyService﹕ onDestroy
```

此时，service先进行解绑，然后进行销毁。被启动又被绑定的服务的生命周期：如果一个Service又被启动又被绑定，则该Service将会一直在后台运行。onCreate始终只会调用一次，startService调用次数与Service的onStart调用次数相同。单独调用unbindService或stopService 或 Service的 stopSelf 来停止服务是无法完成的，必须二者都进行调用。

至此，我们了解了service的启动和停止以及service的生命周期，我们还是总结下。<br>
1、在我们的应用中，停止服务后要处理一些无用的资源，防止占用系统资源。我们在onDestroy()方法中进行释放一些资源。<br>
2、service运行在主线程中，所以如果我们内部有与activity进行交互的操作，不应该占用过度时间，来阻塞activity的执行，防止出现ANR异常。我们应该开启一个子线程进行执行。

* 用户在进行了一种操作后5秒钟没有响应。
* broadCastReceiver所进行的操作在10秒内没有完成。
* Service在20秒内没返回结果。

3、service两种启动方式的生命周期图对比：
![servicelife](http://img.blog.csdn.net/20151014151239304)

* 完整生命周期：从onCreate()方法——>onDestroy()方法。
* 活动生命周期：从 onStartCommand() 或 onBind()开始，如果是start方式启动，这个生命周期段和完整生命周期一样，如果通过bind方式启动，这个生命周期持续到onUnbind方法。

4、activity的销毁重建，会导致service的生命周期重新开始，activity销毁，会自动解绑service。

5、在Manifet中配置service标签的属性：

* android:enabled：标识改service是否可用，true可用，false不可用
* android:exported：别的应用是否可以使用该service，true可以，false不可以。
* android:name：服务的路径名称，必须设置
* android:process：服务运行的进程名称，默认主进程

### 四、Service与Activity的通信
在上面的生命周期测试代码中，在bindService()方法中，我们已经使用了ServiceConnection对象。不错，这个对象就是组件与Service进行通信的媒介，通过ServiceConnection对象，我们能够与service进行交互。在onBind()方法中返回一个IBinder对象，所以我们需要自定义一个Binder的子类来处理业务逻辑，然后返回改Binder子类的实例。我们针对上面的代码修改即可。

创建一个MyBinder的子类：

```
	class MyBinder extends Binder {
        //用于打印Log
        public void showBinderLog(){
            Log.d(TAG,"showBinderLog");
        }
        //获取执行结果
        public int sumNumber(){
            return 12+12;
        }
    }
```

在onBind方法中返回对应的实例：

```
	@Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG,"onBind");
        return new MyBinder();
    }
```

最后在Activity中完善我们的ServiceConnection。

```
	private MyService.MyBinder myBinder;
    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.d(TAG,"onServiceConnected");
            myBinder = (MyService.MyBinder) service;
            myBinder.showBinderLog();
            Log.d(TAG,"Sum=" + myBinder.sumNumber());
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.d(TAG,"onServiceDisconnected");
        }
    };
```

我们看下执行结果：

```
	MyService﹕ onCreate
	MyService﹕ onBind
	MyService:MainActivity﹕ onServiceConnected
	MyService﹕ showBinderLog
	MyService:MainActivity﹕ Sum=24
```

可以看到，我们通过ServiceConnection可以监听到service的状态，然后控制service连接成功后的执行内容，同时获取对应的执行结果，这样，我们就在activity中实现了对service的控制，二者之间通过Binder对象进行通信。

### 五、前台Service
默认情况下，我们启动的service都是后台service，都是在后台为我们处理业务逻辑，但是也有一些前台service，正如前面所说，当系统由于内存原因杀死我们的service时，前台service能够给予用户一种提醒的效果。前台service必须提供notification，同时必须防止"Ongoing"头，这意味着这个通知不能滑动杀出，除非服务停止或者服务从前台移出。常见的：腾讯手机管家、酷狗音乐在运行的时候，顶部的通知栏你就无法滑动删除，就是这个玩意。下面我们来创建一个前台service：

```
	public class MyService extends Service {
		...
		@Override
	    public void onCreate() {
	        super.onCreate();
	        Log.d(TAG,"onCreate");
	        createForegroundService();
	    }
		...

		/**
	     * 创建一个前台服务
	     */
	    private void createForegroundService(){
	        if(Build.VERSION.SDK_INT <= 11){
	            Notification notification = new Notification();
	            notification.icon = R.drawable.god;
	            notification.tickerText="前台服务的实例";
	            notification.when=System.currentTimeMillis();
	            notification.defaults=Notification.DEFAULT_SOUND;
	            notification.setLatestEventInfo(this, "通知", "前台服务",
	                    PendingIntent.getActivity(this, 0, new Intent(this, MainActivity.class), 0));
	            startForeground(1,notification);
	        }else{//体验下高版本的新用法
	            Notification.Builder builder = new Notification.Builder(this);
	            builder.setTicker("前台服务实例");
	            builder.setSmallIcon(R.drawable.god);
	            builder.setWhen(System.currentTimeMillis());
	            builder.setDefaults(Notification.DEFAULT_SOUND);
	            builder.setContentText("这是一个前台服务实例");
	            builder.setContentTitle("前台服务实例");
	            builder.setOngoing(true);
	            builder.setContentIntent(PendingIntent.getActivity(this, 0,
	                            new Intent(this, MainActivity.class), 0));
	            NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
	            if(Build.VERSION.SDK_INT < 16){
	                notificationManager.notify(1,builder.getNotification());
	            }else{
	                notificationManager.notify(1,builder.build());
	            }
	        }
	    }
	}
```

看下效果图：
![foreground](http://img.blog.csdn.net/20151014151346011)

<h3>六、Service与Thread的区别</h3>
这个问题是面试的常客，经常让你说说这二位的区别，所以掌握二者的区别还是很有必要的。我们知道Thread 是程序执行的最小单元，它是分配CPU的基本单位。可以用 Thread 来执行一些异步的操作。 而Service是android系统的四大组件之一，单从这点来说，二者相差甚远。其次，Service是运行在主线程中的，如果我们在Service中处理比较耗时的操作会导致ANR异常。在Service中执行耗时的操作，我们需要另开线程来执行。既然如此，我们为何又要使用Service来执行呢？这个原因就涉及到Android系统对进程的优先级设定。以下是从高到低：

* 1、Foreground process：前台进程，正在交互的
* 2、Visble process：可视进程，可见不能交互
* 3、Service process：服务进程
* 4、Background process：后台进程
* 5、Empty process：空进程

Android系统回收进程的优先级是从低到高，当系统的内存不足的时候，系统依次从低到高来回收进程，注意：系统不会轻易回收服务、可见、前台这三种进程，即使回收了，当系统内存充足的时候，会重新启动这些进程。<br>
明白了这些，基本就懂了为什么service的存在还是很有必要的，当应用的所有界面关闭了，依附在activity运行的线程就成为空进程，很容易被销毁。

<h3>七、Service与Intentservice的区别</h3>
通过查阅文档，我们知道IntentService是Service的子类，我们知道Service执行比较耗时的操作可能导致ANR异常，所以IntentService就弥补了它这个缺点，我们先来看看一个IntentService的案例：

```
	public class MyIntentService extends IntentService {
	    public MyIntentService(){
	        super("MyIntentServices");
	    }
	
	    /**
	     * Creates an IntentService.  Invoked by your subclass's constructor.
	     *
	     * @param name Used to name the worker thread, important only for debugging.
	     */
	    public MyIntentService(String name) {
	        super(name);
	    }
	
	    @Override
	    protected void onHandleIntent(Intent intent) {
	       
	    }
	}
```

我们必须实现onHandleIntent方法来处理我们的Intent，另外还必须声明一个无参的构造函数，不然运行的时候报异常：

	

```
Caused by: java.lang.InstantiationException: can't instantiate class test.drision.com.servicestudy.MyIntentService; no empty constructor
```

通过构造函数的注释，我们知道IntentService的内部创建了一个名为worker thread的线程来处理耗时工作，所以不需要我们在像Service中开启一个线程来处理。为了对比，我们在MyService类的onCreate方法中增加一个线程睡眠：

```
	 @Override
    public void onCreate() {
        super.onCreate();
        try {
            Thread.sleep(20000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

同时在MyIntentService的onHandleIntent中增加同样睡眠20s：

```
	@Override
    protected void onHandleIntent(Intent intent) {
        try {
            Thread.sleep(20000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

最后展现的效果就是：
![ANR](http://img.blog.csdn.net/20151014151436587)
MyService在启动20秒后出现ANR异常，而MyIntentService没有出现。这就是二者最大的差别。

<h3>八、远程Service的使用</h3>
现在说说远程Service的使用。通过上面的学习，我们知道Service与别的组件之间进行通信是通过一个Binder对象，所以远程Service的通信同样是通过Binder进行通信，我们知道远程service的优点就是能为多个进程服务。所以现在我们按照正常的Service来处理下它的通讯，代码如下：

```
	public class RemoteService extends Service {
	    private static final String TAG = "RemoteService";
	    public RemoteService() {
	    }
	
	    @Override
	    public IBinder onBind(Intent intent) {
	        return new MyRemoteBind();
	    }
	
	    public class MyRemoteBind extends Binder{
	        public void showLog(){
	            Log.d(TAG,"showLog");
	        }
	    }
	}
```

看下manifest中的注册：

```
	 <service
            android:name="remoteservice.RemoteService"
            android:enabled="true"
            android:exported="true"
            android:process=":remote">
     </service>
```

我们通过android:process来进行该service运行进程的指定。在MainActivity中我们新增一个Button按钮，来启动该远程service。

```
	 Intent intentRemote = new Intent(MainActivity.this,RemoteService.class);
     bindService(intentRemote,serviceConnectionRemote,BIND_AUTO_CREATE);

	 private RemoteService.MyRemoteBind remoteBind;
     private ServiceConnection serviceConnectionRemote = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            remoteBind = (RemoteService.MyRemoteBind) service;
            remoteBind.showLog();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
```

我们运行看下界面：
![remoteService](http://img.blog.csdn.net/20151014151525284)

擦了，怎么崩了啊！确实崩了，我们看下Log日志：

```
	java.lang.ClassCastException: android.os.BinderProxy cannot be cast to remoteservice.RemoteService$MyRemoteBind
```

根据Log的提示，我们现在无法进行强制转换，这就导致我们按照原来的方式无法与远程service进行通信，所以此时此刻，我们需要找到一种替代方案，即AIDL。

AIDL全称 Android Interface definition language（安卓接口定义语言），它是一种android内部进程通信接口的描述语言，通过它我们可以定义进程间的通信接口，从而实现跨进程通信。

明白了上面的原理，我们下面就来看看AIDL怎么用，在Eclipse下创建一个AIDL文件相当费事，还要改文件后缀名，由于我使用的是Android Studio，所以这里直接有新建AIDL。看看效果图：
![newAIDL](http://img.blog.csdn.net/20151014151551656)

点击完成后，AS会自动给我们生成一个aidl文件下，下面存放我们的文件。
![aidlFile](http://img.blog.csdn.net/20151014151607538)

看下代码：

```
	package remoteservice;

	interface IMyAidlInterface {
	    void showRemoteAIDL();
	}
```

我们仅仅定义一个方法，在Eclipse中当我们定义好一个AIDL，ADT插件会自动在gen目录下给我们生成一个对应的java文件，<font color="#ff0000">在AS中，我们需要在Build菜单下进行Make Project然后才能生成。</font>在Project结构下查看：
![asAIDLStruce](http://img.blog.csdn.net/20151014151634760)

我们打开IMyAidlInterface这个文件看下，这个文件是系统自动针对我们创建的AIDL文件进行生成。

```
	/*
	 * This file is auto-generated.  DO NOT MODIFY.
	 * Original file: E:\\AndroidStudioProject\\FirstTestApp\\servicestudy\\src\\main\\aidl\\remoteservice\\IMyAidlInterface.aidl
	 */
	package remoteservice;
	public interface IMyAidlInterface extends android.os.IInterface
	{
	/** Local-side IPC implementation stub class. */
	public static abstract class Stub extends android.os.Binder implements remoteservice.IMyAidlInterface
	{
	private static final java.lang.String DESCRIPTOR = "remoteservice.IMyAidlInterface";
	/** Construct the stub at attach it to the interface. */
	public Stub()
	{
	this.attachInterface(this, DESCRIPTOR);
	}
	/**
	 * Cast an IBinder object into an remoteservice.IMyAidlInterface interface,
	 * generating a proxy if needed.
	 */
	public static remoteservice.IMyAidlInterface asInterface(android.os.IBinder obj)
	{
	if ((obj==null)) {
	return null;
	}
	android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
	if (((iin!=null)&&(iin instanceof remoteservice.IMyAidlInterface))) {
	return ((remoteservice.IMyAidlInterface)iin);
	}
	return new remoteservice.IMyAidlInterface.Stub.Proxy(obj);
	}
	@Override public android.os.IBinder asBinder()
	{
	return this;
	}
	@Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
	{
	switch (code)
	{
	case INTERFACE_TRANSACTION:
	{
	reply.writeString(DESCRIPTOR);
	return true;
	}
	case TRANSACTION_showRemoteAIDL:
	{
	data.enforceInterface(DESCRIPTOR);
	this.showRemoteAIDL();
	reply.writeNoException();
	return true;
	}
	}
	return super.onTransact(code, data, reply, flags);
	}
	private static class Proxy implements remoteservice.IMyAidlInterface
	{
	private android.os.IBinder mRemote;
	Proxy(android.os.IBinder remote)
	{
	mRemote = remote;
	}
	@Override public android.os.IBinder asBinder()
	{
	return mRemote;
	}
	public java.lang.String getInterfaceDescriptor()
	{
	return DESCRIPTOR;
	}
	@Override public void showRemoteAIDL() throws android.os.RemoteException
	{
	android.os.Parcel _data = android.os.Parcel.obtain();
	android.os.Parcel _reply = android.os.Parcel.obtain();
	try {
	_data.writeInterfaceToken(DESCRIPTOR);
	mRemote.transact(Stub.TRANSACTION_showRemoteAIDL, _data, _reply, 0);
	_reply.readException();
	}
	finally {
	_reply.recycle();
	_data.recycle();
	}
	}
	}
	static final int TRANSACTION_showRemoteAIDL = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
	}
	public void showRemoteAIDL() throws android.os.RemoteException;
	}
```

系统自动生成，代码排版不是很规范，我就挑几点说明下：<br>
![aidlStubStructure](http://img.blog.csdn.net/20151014151711924)
1、根据该类注释，该类是系统自动生成，不允许我们修改。<br>
2、根据该类的定义

```
	public static abstract class Stub extends android.os.Binder implements remoteservice.IMyAidlInterface
```

，该类是一个抽象类，实现了我们定义的AIDL接口，同时是Binder的子类。并且是IMyAidlInterface接口的一个静态内部类。<br>
3、asBinder()方法，返回的是一个Binder对象。<br>
4、编写Aidl文件时，需要注意下面几点: 

* 接口名和aidl文件名相同。
* 接口和方法前不用加访问权限修饰符public,private,protected等,也不能用final,static。
* Aidl默认支持的类型包话java基本类型（int、long、boolean等）和（String、List、Map、CharSequence），使用这些类型时不需要import声明。对于List和Map中的元素类型必须是Aidl支持的类型。如果使用自定义类型作为参数或返回值，自定义类型必须实现Parcelable接口。
* 自定义类型和AIDL生成的其它接口类型在aidl描述文件中，应该显式import，即便在该类和定义的包在同一个包中。
* 在aidl文件中所有非Java基本类型参数必须加上in、out、inout标记，以指明参数是输入参数、输出参数还是输入输出参数。
* Java原始类型默认的标记为in,不能为其它标记。

通过以上的分析，我们基本知道要通过该Stub类进行我们的通讯了，下面我们来进行吧！我们直接在我们的RemoteService类中进行修改。我们直接建立一个Stub的对象，进行使用。

```
	public class RemoteService extends Service {
	    private static final String TAG = "RemoteService";
	    public RemoteService() {
	    }
	
	    @Override
	    public IBinder onBind(Intent intent) {
	        return myRemoteBinder;
	    }
	
	    IMyAidlInterface.Stub myRemoteBinder = new IMyAidlInterface.Stub() {
	        @Override
	        public void showRemoteAIDL() throws RemoteException {
	            Log.d(TAG, "这是通过AIDL进行通讯");
	        }
	    };
	}
```

我们修改下ServiceConnection对象：

```
	private IMyAidlInterface iMyAidlInterface;
    private ServiceConnection serviceConnectionRemote = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service);
            try {
                iMyAidlInterface.showRemoteAIDL();
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
```

这样我们点击Remote Service按钮，即可进行通信使用，看下运行日志结果：
![result1](http://img.blog.csdn.net/20151014151810330)

至此，我们就完成了远程服务的跨进程通信。但是回头想想，我们在同一个项目中使用远程服务，并没有发挥远程服务的威力啊！所以我们创建一个工程来测试远程服务的调用。

创建名称为servicetest的module，将上面servicestudy下的aidl文件夹连同下面的文件一起拷贝到test工程中，如下图：
![same](http://img.blog.csdn.net/20151014151828640)

<font color="#ff0000">注意：此时我们需要在Build菜单下进行make project才能进行使用，这样才能在build文件夹下生成对应的文件供我们使用。</font>这样我们就可以按照上面的步骤进行使用远程服务了。

然后我们在xml文件中创建一个按钮：

```
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">

    <Button
        android:id="@+id/btn_bindRemoteService"
        android:text="BindRemoteService"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="10dp"/>

	</RelativeLayout>
```

在MainActivity中进行处理：

```
	public class MainActivity extends Activity {
	    private Button btn_bindService;
	    private IMyAidlInterface iMyAidlInterface;
	    private ServiceConnection serviceConnection = new ServiceConnection() {
	        @Override
	        public void onServiceConnected(ComponentName name, IBinder service) {
	            iMyAidlInterface = IMyAidlInterface.Stub.asInterface(service);
	            try {
	                iMyAidlInterface.showRemoteAIDL();
	            } catch (RemoteException e) {
	                e.printStackTrace();
	            }
	        }
	
	        @Override
	        public void onServiceDisconnected(ComponentName name) {
	
	        }
	    };
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        btn_bindService = (Button) findViewById(R.id.btn_bindRemoteService);
	        btn_bindService.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                Intent intent = new Intent("test.dsw.com.remoteservice.RemoteService");
	                bindService(intent,serviceConnection,BIND_AUTO_CREATE);
	            }
	        });
	    }
	}
```

看下运行结果：
![result2](http://img.blog.csdn.net/20151014151859493)

至此，我们完成了Service的跨进程通信。这次在android studio中开发的，还是遇到点小麻烦的，不如eclipse中拷包来的方面，主要记住创建aidl文件了一定要make project下，然后才能正常使用。

从昨天写这篇文章开始，中途几次想放弃，感觉Service太庞大了，感觉怎么写都写不全面，参考官方的英文文档，感觉又有很多条例不清，唉！如果有机会，后面有机会写个音乐播放器结合service的使用。欢迎大家留言探讨。

[源码下载地址](http://download.csdn.net/detail/mr_dsw/9180217)

作者：mr_dsw 欢迎转载，与人分享是进步的源泉！

转载请保留地址：http://blog.csdn.net/mr_dsw
