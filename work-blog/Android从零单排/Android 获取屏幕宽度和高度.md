## Android 获取屏幕宽度和高度

在  Android 中获取屏幕宽度和高度的方法大致分为两种，一种是通过 `WindowManager` 读取屏幕宽高，另一种是通过 `Resource` 读取屏幕宽高。

**通过 WindowManager 读取宽高**

```java
public void getWHByWindowManager(Context context) {
    WindowManager windowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
    Display display = windowManager.getDefaultDisplay();
    DisplayMetrics displayMetrics = new DisplayMetrics();
    display.getMetrics(displayMetrics);
    Log.d("Andoter", "height:" + displayMetrics.heightPixels + ",width:" + displayMetrics.widthPixels);
}
```

**通过 Resource 读取宽高**

```java
public void getHWByResource(Context context) {
    DisplayMetrics displayMetrics = context.getResources().getDisplayMetrics();
    Log.d("Andoter", "height:" + displayMetrics.heightPixels + ",width:" + displayMetrics.widthPixels);
}
```

两种方式都可以获取屏幕宽高，那有什么区别呢？

#### WindowManager 获取屏幕宽高

通过 `WindowManager` 获取屏幕宽高时，会首先获取 `Display` 对象，然后获取对应的 `DisplayMetrics` 对象，读取出对应的宽高。

`WindowManager` 是一个用于同 `Window` 交互的接口，通常一个 `WindowManager` 会和对应的 `Display` 对应绑定。

`Display` 对象提供显示区域的 `size` 和 `density` 信息。显示区域有两种描述方式：



- 应用程序显示区域，不包括状态栏(刘海屏，水滴屏等)、导航栏等系统 UI 所占用的空间。通常是比实际的显示区域要小。
- 实际显示区域，包括状态栏(刘海屏，水滴屏等)、导航栏等系统 UI 所占用的空间。尽管如此，如果使用 `adb shell wm size` 指令修改显示区域大小，实际显示区域还是会小于实际物理尺寸大小。

#### Resource 获取屏幕宽高

```java
/**
 * Return the current display metrics that are in effect for this resource 
 * object.  The returned object should be treated as read-only.
 * 
 * @return The resource's current display metrics. 
 */
public DisplayMetrics getDisplayMetrics() {
    return mResourcesImpl.getDisplayMetrics();
}
```

获取对当前显示有影响的 `display metrics`。查看源码可以看到获取的都是与设备相关的信息。



#### 结论

Context.getResources().getDisplayMetrics() 依赖于手机系统，获取到的是系统的屏幕信息；

WindowManager.getDefaultDisplay().getMetrics(dm) 是获取到 Activity 的实际屏幕信息。

