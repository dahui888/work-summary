### Android视图绘制View相关概念基本总结

#### 一、Android系统中View视图坐标系
作者向你抛出一张图：

![coridnate](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/cordinate.png)

注：图片出处作者刘望舒

#### 二、View中的scrollTo和scrollBy

首先我们看下scrollTo方法设置view的滑动距离，来看看的源码：

```java
/**
 * The offset, in pixels, by which the content of this view is scrolled
 * horizontally.
 * {@hide}
 */
@ViewDebug.ExportedProperty(category = "scrolling")
protected int mScrollX;
/**
 * The offset, in pixels, by which the content of this view is scrolled
 * vertically.
 * {@hide}
 */
@ViewDebug.ExportedProperty(category = "scrolling")
protected int mScrollY;

/**
 * Set the scrolled position of your view. This will cause a call to
 * {@link #onScrollChanged(int, int, int, int)} and the view will be
 * invalidated.
 * @param x the x position to scroll to
 * @param y the y position to scroll to
 */
public void scrollTo(int x, int y) {
    if (mScrollX != x || mScrollY != y) {
        int oldX = mScrollX;
        int oldY = mScrollY;
        mScrollX = x;
        mScrollY = y;
        invalidateParentCaches();
        onScrollChanged(mScrollX, mScrollY, oldX, oldY);
        if (!awakenScrollBars()) {
            postInvalidateOnAnimation();
        }
    }
}

/**
 * Move the scrolled position of your view. This will cause a call to
 * {@link #onScrollChanged(int, int, int, int)} and the view will be
 * invalidated.
 * @param x the amount of pixels to scroll by horizontally
 * @param y the amount of pixels to scroll by vertically
 */
public void scrollBy(int x, int y) {
    scrollTo(mScrollX + x, mScrollY + y);
}
```

我们看到，在scrollTo执行的时候，先判断mScrollX和mScrollY是否发生改变。如果发生改变，就触发**View的onScrollChanged方法**。从这里我们也看可以看出**mScrollX和mScrollY分别代表View中的内容的偏移距离。这个距离等于View的左边缘距离内容左边缘的距离或View的上边缘距离内容上边缘的距离。同时在scrollTo方法中也可以看到该值就是我们传入的x，y值。**

<font color="red">这里，我想通过查看onScrollChanged的源码来看看计算的关系规则，但是没有找到这段源码。</font>所以我在网上发了一个帖子[View onScrollChanged](https://stackoverflow.com/questions/45239967/view-onscrollchanged)。同时查看了下面两个文章：[Android View的scroll方法及属性](http://www.jianshu.com/p/bb64a7bfd8ba)、[ Android滑屏 mScrollX mScrollY scrollTo() scrollBy()](http://blog.csdn.net/tuhuolong/article/details/7954080)。但是感觉跟网上看到很多说出来的结论差不多。都是说滑动的是View的内容。由于我也没有找到这个关系计算实现的源码，综合这两个文章，说下理解：

- Android中的坐标系可以抽象为两个：视图坐标（无边界）、布局坐标（可见，有边界）。调用scrollTo方法滑动的是视图坐标系，布局坐标系没有改变。
- 调用scrollTo方法计算偏移量，每次以初始偏移量(0,0)作为基准。比如你视图偏移(100,100)。那你的偏移量就是(-100,-100)，所以调用scrollTo(-100, -100)即可。这样我们理解scrollTo方法(滑动到指定位置)和mScrollX（view内容在X方向上的偏移量）的注释就好理解一些，一切参照都是(0，0)。

#### 三、ViewGroup的事件分发拦截
在Android开发中我们常见的一个问题就是嵌套滑动的控件组合在一起没效果了，这里面就涉及到Android中的事件分发拦截。首先我们需要明确以下几点：
1. Android中的事件分发先传递到ViewGroup，然后在传递到View。
2. ViewGroup通过onInterceptTouchEvent方法对事件拦截。返回值本质上有2种，①返回true：拦截事件，即将事件拦截给自己的onTouchEvent进行处理，不分发给子View。②返回false：不拦截事件，即将事件分发给子View的dispatchTouchEvent进行处理。

下面来简要看看各个方法的源码：

**onInterceptTouchEvent**

```java
/**
 * Implement this method to intercept all touch screen motion events.  This
 * allows you to watch events as they are dispatched to your children, and
 * take ownership of the current gesture at any point.
 *
 * <p>Using this function takes some care, as it has a fairly complicated
 * interaction with {@link View#onTouchEvent(MotionEvent)
 * View.onTouchEvent(MotionEvent)}, and using it requires implementing
 * that method as well as this one in the correct way.  Events will be
 * received in the following order:
 *
 * <ol>
 * <li> You will receive the down event here.
 * <li> The down event will be handled either by a child of this view
 * group, or given to your own onTouchEvent() method to handle; this means
 * you should implement onTouchEvent() to return true, so you will
 * continue to see the rest of the gesture (instead of looking for
 * a parent view to handle it).  Also, by returning true from
 * onTouchEvent(), you will not receive any following
 * events in onInterceptTouchEvent() and all touch processing must
 * happen in onTouchEvent() like normal.
 * <li> For as long as you return false from this function, each following
 * event (up to and including the final up) will be delivered first here
 * and then to the target's onTouchEvent().
 * <li> If you return true from here, you will not receive any
 * following events: the target view will receive the same event but
 * with the action {@link MotionEvent#ACTION_CANCEL}, and all further
 * events will be delivered to your onTouchEvent() method and no longer
 * appear here.
 * </ol>
 *
 * @param ev The motion event being dispatched down the hierarchy.
 * @return Return true to steal motion events from the children and have
 * them dispatched to this ViewGroup through onTouchEvent().
 * The current target will receive an ACTION_CANCEL event, and no further
 * messages will be delivered here.
 */
public boolean onInterceptTouchEvent(MotionEvent ev) {
    if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
            && ev.getAction() == MotionEvent.ACTION_DOWN
            && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
            && isOnScrollbarThumb(ev.getX(), ev.getY())) {
        return true;
    }
    return false;
}
```

注释讲解的很详细，代码比较精简。简要翻译理解下注释中的内容：实现onInterceptTouchEvent方法可以对屏幕的点击事件进行拦截操作，同时能够帮助你在任何时候控制事件的分发。在使用这个方法的时候需要注意与onTouchEvent的交互，交互的原则如下：
你会在onInterceptTouchEvent方法中接收到ACTION_DOWN事件，这个点击事件可能被你的子View处理也可能被ViewGroup的onToucheEvent处理。如果onInterceptTouchEvent返回true，这也意味着你要实现你的onTouchEvent并且返回true，然后你才可以接收后面的Action事件（如果返回false，会将这个事件回调给它的“parent view”）。于此同时，你在onTouchEvent中返回true，你将不会受到onInterceptTouchEvent的任何事件，所有的处理都由dispatchTouchEvent直接分发给onTouchEvent中进行处理。如果你在onInterceptTouchEvent中返回false，即不进行事件拦截，会将事件分发给子view的dispatchTouchEvent进行分发，然后决定是否分发给子view的onTouchEvent是否处理。
q
示例验证：

自定义一个LinearLayout控件：

```java
public class MyLinearLayout extends LinearLayout {

	private static final String TAG = "MyLinearLayout";

	public MyLinearLayout(Context context, AttributeSet attrs) {
		super(context, attrs);
	}

	@Override
	public boolean onInterceptTouchEvent(MotionEvent ev) {
		Log.d(TAG, "onInterceptTouchEvent");
		return super.onInterceptTouchEvent(ev);
	}

	@Override
	public boolean onTouchEvent(MotionEvent event) {
		Log.d(TAG, "onTouchEvent");
		return super.onTouchEvent(event);
	}
	
	@Override
	public boolean dispatchTouchEvent(MotionEvent event) {
		Log.d(TAG, "dispatchTouchEvent");
		return super.dispatchTouchEvent(event);
	}
}
```
自定义一个Button控件：
```java
public class MyButton extends Button {
	private static final String TAG = "MyLinearLayout";
	public MyButton(Context context, AttributeSet attrs) {
		super(context, attrs);
	}

	@Override
	public boolean onTouchEvent(MotionEvent event) {
		Log.d(TAG, "onTouchEvent:child");
		return super.onTouchEvent(event);
	}

	@Override
	public boolean dispatchTouchEvent(MotionEvent event) {
		Log.d(TAG, "dispatchTouchEvent:child");
		return super.dispatchTouchEvent(event);
	}
}
```

这样我们简单布局下就可以验证了。

##### onInterceptTouchEvent事件

 **1、onInterceptTouchEvent返回true，拦截事件。**

```java
07-26 12:53:08.063: D/MyLinearLayout(4190): dispatchTouchEvent
07-26 12:53:08.063: D/MyLinearLayout(4190): onInterceptTouchEvent
07-26 12:53:08.063: D/MyLinearLayout(4190): onTouchEvent
```
事件执行的顺序是dispatchTouchEvent->onIntercetTouchEvent->onTouchEvent。
即，**当onInterceptTouchEvent返回true拦截事件的时候，会交由自己的onTouchEvent进行处理。**

**2、onInterceptTouchEvent返回false，不拦截事件。**
```java
07-26 18:52:57.920: D/MyLinearLayout(3368): dispatchTouchEvent
07-26 18:52:57.920: D/MyLinearLayout(3368): onInterceptTouchEvent
07-26 18:52:57.920: D/MyLinearLayout(3368): dispatchTouchEvent:child
07-26 18:52:57.920: D/MyLinearLayout(3368): onTouchEvent:child
```
事件的执行顺序是dispatchTouchEvent->onInterceptTouchEvent->dispatchTouchEvent:child->onTouchEvent:child。**即不拦截事件的时候，会将事件分发给子View的dispatchTouchEvent，然后由它决定是否分发给子View的onTouchEvent进行处理。**


#####  onTouchEvent事件

**1、onTouchEvent返回true**
执行操作：点击——>拖动
```java
07-26 12:53:08.063: D/MyLinearLayout(4190): dispatchTouchEvent
07-26 12:53:08.063: D/MyLinearLayout(4190): onInterceptTouchEvent
07-26 12:53:08.063: D/MyLinearLayout(4190): onTouchEvent
07-26 12:53:10.195: D/MyLinearLayout(4190): dispatchTouchEvent
07-26 12:53:10.195: D/MyLinearLayout(4190): onTouchEvent
07-26 12:53:11.547: D/MyLinearLayout(4190): dispatchTouchEvent
07-26 12:53:11.547: D/MyLinearLayout(4190): onTouchEvent

```
通过上面可以得出结论：**当onTouchEvent返回true的时候表示消费处理此次屏幕单击事件中所有的action，在下次点击事件发生前（下次MotionEvent.ACTION_DOWN前），所有的操作都由dispatchTouchEvent直接分发给onTouchEvent事件进行处理。**


**2、onTouchEvent返回false**
执行操作：点击——>拖动
```java
07-26 18:50:00.934: D/MyLinearLayout(2572): dispatchTouchEvent
07-26 18:50:00.934: D/MyLinearLayout(2572): onInterceptTouchEvent
07-26 18:50:00.934: D/MyLinearLayout(2572): dispatchTouchEvent:child
07-26 18:50:00.934: D/MyLinearLayout(2572): onTouchEvent:child
07-26 18:50:00.934: D/MyLinearLayout(2572): onTouchEvent
```
通过上面我们可以得出如下结论：**当onTouchEvent返回false的时候，意味着不消费此次的屏幕点击事件。所以后面拖动触发的ACTION_MOVE事件也不会分发给onTouchEvent进行处理。而是将事件返回给parent view进行消费处理。**

注意：
1. super.onInterceptTouchEvent默认是不拦截事件；
2. super.onTouchEvent默认是处理事件
3. super.dispatchTouchEvent默认是分发
4. 当时间分发到子view时，如果子view不分发或不处理。皆返回给parent view的onTouchEvent进行处理。

#### 四、Scroller工具类的使用。
通过上面的介绍，我们知道scrollTo和scrollBy都是瞬时操作，在一些场景，当用户抬起动作的时候，会执行缓慢的滑动。这个时候就利用到了Scroller类。Scroller类的作用就是可以根据我们传入的位置信息，计算位置。然后我们在配合View的computeScroll方法进行页面的滚动。

简单的Scroller使用

```java
private Scroller mScroller = new Scroller(context);
mScroller.forceFinished(true);
mScroller.startScroll(0, 0, 100, 0);
invalidate();
```
常见方法解析：
1、Scroller.computeScrollOffset()
```java
/**
 * Call this when you want to know the new location.  If it returns true,
 * the animation is not yet finished.
 */ 
public boolean computeScrollOffset() {
    if (mFinished) {
        return false;
    }

    int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);

    if (timePassed < mDuration) {
        switch (mMode) {
        case SCROLL_MODE:
            final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);
            mCurrX = mStartX + Math.round(x * mDeltaX);
            mCurrY = mStartY + Math.round(x * mDeltaY);
            break;
        case FLING_MODE:
            final float t = (float) timePassed / mDuration;
            final int index = (int) (NB_SAMPLES * t);
            float distanceCoef = 1.f;
            float velocityCoef = 0.f;
            if (index < NB_SAMPLES) {
                final float t_inf = (float) index / NB_SAMPLES;
                final float t_sup = (float) (index + 1) / NB_SAMPLES;
                final float d_inf = SPLINE_POSITION[index];
                final float d_sup = SPLINE_POSITION[index + 1];
                velocityCoef = (d_sup - d_inf) / (t_sup - t_inf);
                distanceCoef = d_inf + (t - t_inf) * velocityCoef;
            }

            mCurrVelocity = velocityCoef * mDistance / mDuration * 1000.0f;

            mCurrX = mStartX + Math.round(distanceCoef * (mFinalX - mStartX));
            // Pin to mMinX <= mCurrX <= mMaxX
            mCurrX = Math.min(mCurrX, mMaxX);
            mCurrX = Math.max(mCurrX, mMinX);

            mCurrY = mStartY + Math.round(distanceCoef * (mFinalY - mStartY));
            // Pin to mMinY <= mCurrY <= mMaxY
            mCurrY = Math.min(mCurrY, mMaxY);
            mCurrY = Math.max(mCurrY, mMinY);

            if (mCurrX == mFinalX && mCurrY == mFinalY) {
                mFinished = true;
            }

            break;
        }
    }
    else {
        mCurrX = mFinalX;
        mCurrY = mFinalY;
        mFinished = true;
    }
    return true;
}
```
当我们获取新位置的时候，通过这个方法判断滑动是否仍在进行中。如果返回true，表示还没结束，我们可以通过Scroller获取位置。

2、isFinished()：判断Scroll的scroll是否结束，返回true表示结束。

3、Scroller默认的duration是250ms。


#### 五、通过上面对View的几个基础概念分析，我们写个小Demo。滑动的LinearLayout。
```java
/**
 * Created by Iflytek_dsw on 2017/7/24.
 */

public class ScrollLinearLayout extends LinearLayout {
    private static final String TAG = "ScrollLinearLayout";
    private ScrollOrientation scrollOrientation = ScrollOrientation.HORIZONTAL;
    private float mXDown,mYDown;
    private float mXMove,mYMove,mXLastMove,mYLastMove;
    private int touchSlop;
    private Scroller mScroller;
    private int leftBorder, rightBorder;
    private int topBorder, bottomBorder;
    public enum ScrollOrientation{
        VERTICAL,
        HORIZONTAL,
    }

    public ScrollLinearLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        touchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
        mScroller = new Scroller(context);
        Log.d(TAG, "TouchSlop:" + touchSlop);
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);
        int childCount = getChildCount();
        if(scrollOrientation == ScrollOrientation.HORIZONTAL){
            for(int index = 0; index < childCount; index++){
                View childView = getChildAt(index);
                childView.layout(l + r*index, t, r + r*index, b);
            }
        }else if(scrollOrientation == ScrollOrientation.VERTICAL){
            for(int index = 0; index < childCount; index++){
                View childView = getChildAt(index);
                childView.layout(l, t+ b * index, r, b + b * index);
            }
        }
        int lindex = 0;
        int rindex = getChildCount() -2;
        leftBorder = getChildAt(lindex).getLeft();
        rightBorder = rindex > 0 ? getChildAt(rindex).getRight() : getChildAt(lindex).getRight();
        topBorder = getChildAt(0).getTop();
        bottomBorder = rindex > 0 ? getChildAt(rindex).getBottom() : getChildAt(lindex).getBottom();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        if(widthMode != MeasureSpec.EXACTLY){
            widthSize = 300;
        }

        if(heightMode != MeasureSpec.EXACTLY){
            heightSize = 200;
        }
        setMeasuredDimension(widthSize, heightSize);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.d(TAG,"onInterceptTouchEvent");
        int code = ev.getAction();
        switch (code){
            case MotionEvent.ACTION_DOWN:
                mXDown = ev.getRawX();
                mYDown = ev.getRawY();
                mXLastMove = mXDown;
                mYLastMove = mYDown;
                Log.d(TAG,"downX:" + mXDown + ",downY:" + mYDown);
                break;
            case MotionEvent.ACTION_MOVE:
                mXMove = ev.getRawX();
                mYMove = ev.getRawY();
                Log.d(TAG,"moveX:" + mXMove + ",moveY:" + mYMove);
                Log.d(TAG,"moveLastX:" + mXLastMove + ",moveLastY:" + mYLastMove);
                if(scrollOrientation == ScrollOrientation.HORIZONTAL){
                    if(Math.abs(mXMove - mXLastMove) > touchSlop){
                        mXLastMove = mXMove;
                        return true;
                    }
                }else if(scrollOrientation == ScrollOrientation.VERTICAL){
                    if(Math.abs(mYMove - mYLastMove) > touchSlop){
                        mYLastMove = mYMove;
                        return true;
                    }
                }
                break;
        }
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d(TAG,"onTouchEvent");
        int keyCode = event.getAction();
        switch (keyCode){
            case MotionEvent.ACTION_MOVE:
                mXMove = event.getRawX();
                mYMove = event.getRawY();
                if(scrollOrientation == ScrollOrientation.HORIZONTAL){
                    int scrollX = (int) (mXLastMove - mXMove);
                    Log.d(TAG,"mScrollX=" + getScrollX() + " --  scrollX=" + scrollX);
                    Log.d(TAG,"leftBorder=" + leftBorder + " --  rightBorder=" + rightBorder);
                    //判断限制左右边界
                    if(getScrollX() + scrollX < leftBorder){
                        /**
                         * 如果已经在左边缘，在继续向右滑动，scrollX<0
                         */
                        break;
                    }else if(getScrollX() + scrollX > rightBorder){
                        /**
                         * 如果已经在右边缘，如果继续向左滑动已经超出右边缘范围，向左滑动。mScrollX>0.
                         */
                        break;
                    }
                    scrollBy(scrollX, 0);
                    mXLastMove = mXMove;
                }else if(scrollOrientation == ScrollOrientation.VERTICAL){
                    int scrollY = (int) (mYLastMove - mYMove);
                    if(getScrollY() + scrollY < topBorder ){
                        break;
                    }else if(getScrollY() + scrollY > bottomBorder){
                        break;
                    }

                    scrollBy(0, scrollY);
                    mYLastMove = mYMove;
                }
                break;
            case MotionEvent.ACTION_UP:
                //滑动超过一半就滚动到上一个页面或下一个页面
                if(scrollOrientation == ScrollOrientation.HORIZONTAL){
                    int pageIndex = (getScrollX() + getWidth() / 2) / getWidth();
                    int dx = pageIndex * getWidth() -getScrollX();
                    mScroller.startScroll(getScrollX(), 0, dx, 0, 500);
                    invalidate();
                }else if(scrollOrientation == ScrollOrientation.VERTICAL){
                    int pageIndex = (getScrollY() + getHeight() / 2) / getHeight();
                    int dy = pageIndex * getHeight() - getScrollY();
                    mScroller.startScroll(0, getScrollY(), 0, dy, 500);
                    invalidate();
                }
                break;
        }
        return super.onTouchEvent(event);
    }

    @Override
    public void computeScroll() {
        if(mScroller.computeScrollOffset()){
            scrollTo(mScroller.getCurrX(),mScroller.getCurrY());
            invalidate();
        }
    }

    /**
     * 获取滑动方向
     * @return
     */
    public ScrollOrientation getScrollOrientation() {
        return scrollOrientation;
    }

    /**
     * 设置滑动方向
     * @param scrollOrientation
     */
    public void setScrollOrientation(ScrollOrientation scrollOrientation) {
        this.scrollOrientation = scrollOrientation;
        forceLayout();
        requestLayout();
        invalidate();
    }
}
```
![scroll](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/circle.gif)
