## Android Toast基础与原理

### 一、Toast的使用方式

1. Toast.makeText(context,text,duration)
2. public Toast(Context context)

在Android系统中，给我们提供了两种方式来创建一个Toast对象。第一种是通过makeText方法快速构建Toast对象。第二种是通过Toast的构造方法进行创造一个空的（不含View）的Toast对象。注意，通过构造方法创建的Toast对象，在show()前需要我们调用setView（View view）方法来设置显示的View。

这里我们简要查看下makeText方法和构造方法的源码。

##### 构造方法
```java
/**
 * Construct an empty Toast object.  You must call {@link #setView} before you
 * can call {@link #show}.
 *
 * @param context  The context to use.  Usually your {@link android.app.Application}
 *                 or {@link android.app.Activity} object.
 */
public Toast(Context context) {
    mContext = context;
    mTN = new TN();
    mTN.mY = context.getResources().getDimensionPixelSize(
            com.android.internal.R.dimen.toast_y_offset);
    mTN.mGravity = context.getResources().getInteger(
            com.android.internal.R.integer.config_toastDefaultGravity);
}
```
在上面的源码中，我们可以看到构造方法中核心操作是创建一个TN对象，并给它设置默认的y轴方向上的偏移和默认的对齐方式Gravity。

##### makeText方法
```java
/**
 * Make a standard toast that just contains a text view.
 */
public static Toast makeText(Context context, CharSequence text, @Duration int duration) {
    Toast result = new Toast(context);

    LayoutInflater inflate = (LayoutInflater)
            context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    View v = inflate.inflate(com.android.internal.R.layout.transient_notification, null);
    TextView tv = (TextView)v.findViewById(com.android.internal.R.id.message);
    tv.setText(text);

    result.mNextView = v;
    result.mDuration = duration;

    return result;
}
```
makeText方法会构建一个只含有TextView的视图来展示。源码中第一步还是通过构造函数构建一个空的Toast对象，然后通过LayoutInflater解析一个View对象，绑定到Toast对象上进行展示。

通过上面对makeToast的源码分析，我们比葫芦画瓢的总结下自定义视图Toast的实现过程：

1. 通过构造方法创建一个空的Toast对象；
2. 自定义视图layout，通过LayoutInflate解析成View对象，通过Toast.setView方法设置。
3. 调用Toast.show()方法进行展示。

### 二、Toast常用方法

##### 1、setDuration(@Duration int duration)
设置Toast的显示时长，有两个值：LENGTH_SHORT, LENGTH_LONG。

##### 2、setGravity(int gravity, int xOffset, int yOffset)
设置Toast显示的位置，这里有三个参数。

- gravity：toast显示的位置，CENTER_VERTICAL（垂直居中）、CENTER_HORIZONTAL（水平居中）、TOP（顶部）；
- xOffset：Toast在水平方向（x轴）的偏移量，偏移量单位为，大于0向右偏移，小于0向左偏移；
- yOffset：Toast在垂直方向（y轴）的偏移量，大于0向下偏移，小于0向上偏移；

##### 3、setMargin(float horizontalMargin, float verticalMargin)

##### 4、setText(CharSequence s)
设置Toast的显示文字。

##### 5、setView(View view)
设置Toast展示的View样式。可用于我们的自定义Toast显示样式。

##### 6、show()
Toast的显示调用的方法。

### 三、Toast的展示与管理
通过上面的介绍，我们知道通过Toast.show()方法完成Toast的显示。

```java
/**
 * Show the view for the specified duration.
 */
public void show() {
    if (mNextView == null) {
        throw new RuntimeException("setView must have been called");
    }

    INotificationManager service = getService();
    String pkg = mContext.getOpPackageName();
    TN tn = mTN;
    tn.mNextView = mNextView;

    try {
        service.enqueueToast(pkg, tn, mDuration);
    } catch (RemoteException e) {
        // Empty
    }
}
```
通过show的源码可以看到系统中创建一个INotificationManager对象，然后通过enqueueToast方法将一个TN对象入队列。这里，我们需要看看这个TN对象是什么？

##### TN对象是什么？
```java
private static class TN extends ITransientNotification.Stub 
```
又是熟悉的面孔，却是不一样的味道。一个Binder的子类出现了。我们来看看ItransientNotification定义了哪些接口。

```java
/** @hide */
oneway interface ITransientNotification {
    void show();
    void hide();
}
```
显示与隐藏的方法。我们知道了定义的接口方法在去看它的实现类思路会清晰很多，看看TN的实现：

```java
private static class TN extends ITransientNotification.Stub {
    final Runnable mHide = new Runnable() {
        @Override
        public void run() {
            handleHide();
            // Don't do this in handleHide() because it is also invoked by handleShow()
            mNextView = null;
        }
    };

    private final WindowManager.LayoutParams mParams = new WindowManager.LayoutParams();
    final Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            IBinder token = (IBinder) msg.obj;
            handleShow(token);
        }
    };

    int mGravity;
    int mX, mY;
    float mHorizontalMargin;
    float mVerticalMargin;


    View mView;
    View mNextView;
    int mDuration;

    WindowManager mWM;

    static final long SHORT_DURATION_TIMEOUT = 5000;
    static final long LONG_DURATION_TIMEOUT = 1000;

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

    /**
     * schedule handleShow into the right thread
     */
    @Override
    public void show(IBinder windowToken) {
        if (localLOGV) Log.v(TAG, "SHOW: " + this);
        mHandler.obtainMessage(0, windowToken).sendToTarget();
    }

    /**
     * schedule handleHide into the right thread
     */
    @Override
    public void hide() {
        if (localLOGV) Log.v(TAG, "HIDE: " + this);
        mHandler.post(mHide);
    }

    public void handleShow(IBinder windowToken) {
        if (localLOGV) Log.v(TAG, "HANDLE SHOW: " + this + " mView=" + mView
                + " mNextView=" + mNextView);
        if (mView != mNextView) {
            // remove the old view if necessary
            handleHide();
            mView = mNextView;
            Context context = mView.getContext().getApplicationContext();
            String packageName = mView.getContext().getOpPackageName();
            if (context == null) {
                context = mView.getContext();
            }
            mWM = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
            // We can resolve the Gravity here by using the Locale for getting
            // the layout direction
            final Configuration config = mView.getContext().getResources().getConfiguration();
            final int gravity = Gravity.getAbsoluteGravity(mGravity, config.getLayoutDirection());
            mParams.gravity = gravity;
            if ((gravity & Gravity.HORIZONTAL_GRAVITY_MASK) == Gravity.FILL_HORIZONTAL) {
                mParams.horizontalWeight = 1.0f;
            }
            if ((gravity & Gravity.VERTICAL_GRAVITY_MASK) == Gravity.FILL_VERTICAL) {
                mParams.verticalWeight = 1.0f;
            }
            mParams.x = mX;
            mParams.y = mY;
            mParams.verticalMargin = mVerticalMargin;
            mParams.horizontalMargin = mHorizontalMargin;
            mParams.packageName = packageName;
            mParams.hideTimeoutMilliseconds = mDuration ==
                Toast.LENGTH_LONG ? LONG_DURATION_TIMEOUT : SHORT_DURATION_TIMEOUT;
            mParams.token = windowToken;
            if (mView.getParent() != null) {
                if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
                mWM.removeView(mView);
            }
            if (localLOGV) Log.v(TAG, "ADD! " + mView + " in " + this);
            mWM.addView(mView, mParams);
            trySendAccessibilityEvent();
        }
    }

    private void trySendAccessibilityEvent() {
        AccessibilityManager accessibilityManager =
                AccessibilityManager.getInstance(mView.getContext());
        if (!accessibilityManager.isEnabled()) {
            return;
        }
        // treat toasts as notifications since they are used to
        // announce a transient piece of information to the user
        AccessibilityEvent event = AccessibilityEvent.obtain(
                AccessibilityEvent.TYPE_NOTIFICATION_STATE_CHANGED);
        event.setClassName(getClass().getName());
        event.setPackageName(mView.getContext().getPackageName());
        mView.dispatchPopulateAccessibilityEvent(event);
        accessibilityManager.sendAccessibilityEvent(event);
    }        

    public void handleHide() {
        if (localLOGV) Log.v(TAG, "HANDLE HIDE: " + this + " mView=" + mView);
        if (mView != null) {
            // note: checking parent() just to make sure the view has
            // been added...  i have seen cases where we get here when
            // the view isn't yet added, so let's try not to crash.
            if (mView.getParent() != null) {
                if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
                mWM.removeViewImmediate(mView);
            }

            mView = null;
        }
    }
}
```
在TN类的实现内部，实现了show和hide方法。然后通过Handler进行发送消息来实现。最终是通过handleShow和handleHide来完成Toast的显示和取消。通过上面源码可以看到，本质上又是通过WindowManager来进行管理和显示的。


##### INotificationManager怎么创建的？

在这里面通过getService方法获取INotificationManager对象。

```java
private static INotificationManager sService;

static private INotificationManager getService() {
    if (sService != null) {
        return sService;
    }
    sService = INotificationManager.Stub.asInterface(ServiceManager.getService("notification"));
    return sService;
}
```
在这段代码里面，我们看到asInterface这个方法接口，我们就可以根据经验判断这个肯定是INotificationManager是一个AIDL文件。而通过：
```java
ServiceManager.getService("notification")
```
获取的肯定是一个实现该AIDL的Binder对象。
```java
public static IBinder getService(String name) {
    try {
        IBinder service = sCache.get(name);
        if (service != null) {
            return service;
        } else {
            return getIServiceManager().getService(name);
        }
    } catch (RemoteException e) {
        Log.e(TAG, "error in getService", e);
    }
    return null;
}
```

不由得让我们联想到NotificationManager通知管理。我们粗略的查看下NotificationManager的源码可以发现这样一段代码：
```java
private static INotificationManager sService;

/** @hide */
static public INotificationManager getService()
{
    if (sService != null) {
        return sService;
    }
    IBinder b = ServiceManager.getService("notification");
    sService = INotificationManager.Stub.asInterface(b);
    return sService;
}
```
是不是惊奇的发现，这两段代码是惊人的相似。确实，这也可以印证，**Toast和Notification在系统中都是由INotificationManager进行管理完成。**

##### INotificationManager是什么？
INotificationManager是一个AIDL文件，那么它定义了哪些接口呢？
```java
interface INotificationManager
{
    void cancelAllNotifications(String pkg, int userId);
    void enqueueToast(String pkg, ITransientNotification callback, int duration);
    void cancelToast(String pkg, ITransientNotification callback);
    void enqueueNotificationWithTag(String pkg, String opPkg, String tag, int id,
            in Notification notification, inout int[] idReceived, int userId);
    void cancelNotificationWithTag(String pkg, String tag, int id, int userId);

    void setNotificationsEnabledForPackage(String pkg, int uid, boolean enabled);
    boolean areNotificationsEnabledForPackage(String pkg, int uid);

    void setPackagePriority(String pkg, int uid, int priority);
    int getPackagePriority(String pkg, int uid);

    void setPackagePeekable(String pkg, int uid, boolean peekable);
    boolean getPackagePeekable(String pkg, int uid);

    void setPackageVisibilityOverride(String pkg, int uid, int visibility);
    int getPackageVisibilityOverride(String pkg, int uid);

    // TODO: Remove this when callers have been migrated to the equivalent
    // INotificationListener method.
    StatusBarNotification[] getActiveNotifications(String callingPkg);
    StatusBarNotification[] getHistoricalNotifications(String callingPkg, int count);

    void registerListener(in INotificationListener listener, in ComponentName component, int userid);
    void unregisterListener(in INotificationListener listener, int userid);

    void cancelNotificationFromListener(in INotificationListener token, String pkg, String tag, int id);
    void cancelNotificationsFromListener(in INotificationListener token, in String[] keys);

    void setNotificationsShownFromListener(in INotificationListener token, in String[] keys);

    ParceledListSlice getActiveNotificationsFromListener(in INotificationListener token, in String[] keys, int trim);
    void requestHintsFromListener(in INotificationListener token, int hints);
    int getHintsFromListener(in INotificationListener token);
    void requestInterruptionFilterFromListener(in INotificationListener token, int interruptionFilter);
    int getInterruptionFilterFromListener(in INotificationListener token);
    void setOnNotificationPostedTrimFromListener(in INotificationListener token, int trim);
    void setInterruptionFilter(String pkg, int interruptionFilter);

    ComponentName getEffectsSuppressor();
    boolean matchesCallFilter(in Bundle extras);
    boolean isSystemConditionProviderEnabled(String path);

    int getZenMode();
    ZenModeConfig getZenModeConfig();
    boolean setZenModeConfig(in ZenModeConfig config, String reason);
    oneway void setZenMode(int mode, in Uri conditionId, String reason);
    oneway void notifyConditions(String pkg, in IConditionProvider provider, in Condition[] conditions);
    oneway void requestZenModeConditions(in IConditionListener callback, int relevance);
    boolean isNotificationPolicyAccessGranted(String pkg);
    NotificationManager.Policy getNotificationPolicy(String pkg);
    void setNotificationPolicy(String pkg, in NotificationManager.Policy policy);
    String[] getPackagesRequestingNotificationPolicyAccess();
    boolean isNotificationPolicyAccessGrantedForPackage(String pkg);
    void setNotificationPolicyAccessGranted(String pkg, boolean granted);

    byte[] getBackupPayload(int user);
    void applyRestore(in byte[] payload, int user);

    ParceledListSlice getAppActiveNotifications(String callingPkg, int userId);
}
```
在上面我们看到了enqueueToast、cancelToast两个方法，这两个方法就是对应我们show里面的方法。那么这两个方法是在哪里实现的呢？

##### INotificationManager的实现
通过查看源码，我们发现INotificationManager中的接口是在NotificationManagerService中进行实现的。
```java
private final IBinder mService = new INotificationManager.Stub() {
1062        // Toasts
1063        // ============================================================================
1064
1065        @Override
1066        public void enqueueToast(String pkg, ITransientNotification callback, int duration)
1067        {
1068            if (DBG) {
1069                Slog.i(TAG, "enqueueToast pkg=" + pkg + " callback=" + callback
1070                        + " duration=" + duration);
1071            }
1072
1073            if (pkg == null || callback == null) {
1074                Slog.e(TAG, "Not doing toast. pkg=" + pkg + " callback=" + callback);
1075                return ;
1076            }
1077
1078            final boolean isSystemToast = isCallerSystem() || ("android".equals(pkg));
1079
1080            if (ENABLE_BLOCKED_TOASTS && !noteNotificationOp(pkg, Binder.getCallingUid())) {
1081                if (!isSystemToast) {
1082                    Slog.e(TAG, "Suppressing toast from package " + pkg + " by user request.");
1083                    return;
1084                }
1085            }
1086
1087            synchronized (mToastQueue) {
1088                int callingPid = Binder.getCallingPid();
1089                long callingId = Binder.clearCallingIdentity();
1090                try {
1091                    ToastRecord record;
1092                    int index = indexOfToastLocked(pkg, callback);
1093                    // If it's already in the queue, we update it in place, we don't
1094                    // move it to the end of the queue.
1095                    if (index >= 0) {
1096                        record = mToastQueue.get(index);
1097                        record.update(duration);
1098                    } else {
1099                        // Limit the number of toasts that any given package except the android
1100                        // package can enqueue.  Prevents DOS attacks and deals with leaks.
1101                        if (!isSystemToast) {
1102                            int count = 0;
1103                            final int N = mToastQueue.size();
1104                            for (int i=0; i<N; i++) {
1105                                 final ToastRecord r = mToastQueue.get(i);
1106                                 if (r.pkg.equals(pkg)) {
1107                                     count++;
1108                                     if (count >= MAX_PACKAGE_NOTIFICATIONS) {
1109                                         Slog.e(TAG, "Package has already posted " + count
1110                                                + " toasts. Not showing more. Package=" + pkg);
1111                                         return;
1112                                     }
1113                                 }
1114                            }
1115                        }
1116
1117                        record = new ToastRecord(callingPid, pkg, callback, duration);
1118                        mToastQueue.add(record);
1119                        index = mToastQueue.size() - 1;
1120                        keepProcessAliveLocked(callingPid);
1121                    }
1122                    // If it's at index 0, it's the current toast.  It doesn't matter if it's
1123                    // new or just been updated.  Call back and tell it to show itself.
1124                    // If the callback fails, this will remove it from the list, so don't
1125                    // assume that it's valid after this.
1126                    if (index == 0) {
1127                        showNextToastLocked();
1128                    }
1129                } finally {
1130                    Binder.restoreCallingIdentity(callingId);
1131                }
1132            }
1133        }
1134
1135        @Override
1136        public void cancelToast(String pkg, ITransientNotification callback) {
1137            Slog.i(TAG, "cancelToast pkg=" + pkg + " callback=" + callback);
1138
1139            if (pkg == null || callback == null) {
1140                Slog.e(TAG, "Not cancelling notification. pkg=" + pkg + " callback=" + callback);
1141                return ;
1142            }
1143
1144            synchronized (mToastQueue) {
1145                long callingId = Binder.clearCallingIdentity();
1146                try {
1147                    int index = indexOfToastLocked(pkg, callback);
1148                    if (index >= 0) {
1149                        cancelToastLocked(index);
1150                    } else {
1151                        Slog.w(TAG, "Toast already cancelled. pkg=" + pkg
1152                                + " callback=" + callback);
1153                    }
1154                } finally {
1155                    Binder.restoreCallingIdentity(callingId);
1156                }
1157            }
1158        }
```

在Toast的显示过程中，我们可以发现最终调用的showNextToastLocked方法进行显示。
```java
    void showNextToastLocked() {
        ToastRecord record = mToastQueue.get(0);
        while (record != null) {
            if (DBG) Slog.d(TAG, "Show pkg=" + record.pkg + " callback=" + record.callback);
            try {
                record.callback.show();
                scheduleTimeoutLocked(record);
                return;
            } catch (RemoteException e) {
               Slog.w(TAG, "Object died trying to show notification " + record.callback
                        + " in package " + record.pkg);
                // remove it from the list and let the process die
                int index = mToastQueue.indexOf(record);
                if (index >= 0) {
                    mToastQueue.remove(index);
                }
                keepProcessAliveLocked(record.pid);
                if (mToastQueue.size() > 0) {
                    record = mToastQueue.get(0);
                } else {
                   record = null;
               }
           }
       }
   }
   
private void scheduleTimeoutLocked(ToastRecord r)
   {
       mHandler.removeCallbacksAndMessages(r);
       Message m = Message.obtain(mHandler, MESSAGE_TIMEOUT, r);
       long delay = r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY;
        mHandler.sendMessageDelayed(m, delay);
    }
```
其中有一段代码：
```java
record.callback.show();
```
这个就是TN中的show方法。所以这样整个显示的过程就打通了。

通过NotificationManager的源码我们可以得到以下信息：
1、Toast的显示时间长短

```java
    static final int LONG_DELAY = 3500; // 3.5 seconds
    static final int SHORT_DELAY = 2000; // 2 seconds
```
2、Toast内部管理是通过List集合的形式进行管理。
```java
final ArrayList<ToastRecord> mToastQueue = new ArrayList<ToastRecord>();
```
3、Toast在一个包中，最多可在Toast队列中添加50个。
```java
static final int MAX_PACKAGE_NOTIFICATIONS = 50;
```
4、Toast内部显示消息管理通过Handler进行实现。


#### 总结
Toast在创建的时候都会创建TN对象。该对象实现跨进程的通信协议show、hide方法。最后通过INotificationManager进行统一的管理。最后通过TN实现的show方法进行展示。