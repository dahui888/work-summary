### 开源项目——ILayoutAnimationController
>在Android开发中，我们都知道系统给我们提供了常见的动画支持功能，比如帧动画、补间动画、属性动画。

今天给大家推荐的是ViewGroup动画的一个开源工具类ILayoutAnimationController。地址：https://github.com/HuanHaiLiuXin/ILayoutAnimationController


首先给大家介绍下ViewGroup动画的实现：使用LayoutAnimationController用于为一个layout里面的控件，或者是一个ViewGroup里面的控件设置动画效果，可以在XML文件中设置，亦可以在Java代码中设置。
![SimpleCode](https://github.com/dengshiwei/work-summary/blob/master/work-blog/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%E2%80%94%E2%80%94ILayoutAnimationController/simpleCode.png)

这样就是一个简单的使用。

#### ILayoutAnimationController介绍

自定义LayoutAnimationController，可任意定制ViewGroup实例内部子View的动画执行顺序，1行代码就让你的ViewGroup拥有华丽的布局动画！

![ic](https://github.com/dengshiwei/work-summary/blob/master/work-blog/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%E2%80%94%E2%80%94ILayoutAnimationController/ILayoutAnimationController%E5%BD%95%E5%B1%8F.gif)

使用方法：

##### 方法一：首先创建ILayoutAnimationController实例，然后将此实例作为参数为ViewGroup设置布局动画

    1：{@link ILayoutAnimationController#generateController(Animation, float, ILayoutAnimationController.IndexAlgorithm)}

    2：{@link android.view.ViewGroup#setLayoutAnimation(LayoutAnimationController)}

##### 方法二：1行代码直接搞定,以下两种方法任选

    {@link ILayoutAnimationController#setLayoutAnimation(ViewGroup, int, float, ILayoutAnimationController.IndexAlgorithm)}

    {@link ILayoutAnimationController#setLayoutAnimation(ViewGroup, Animation, float,ILayoutAnimationController.IndexAlgorithm)}

**示例代码：**

    LinearLayout ll = (LinearLayout) findViewById(R.id.ll);
    //两行代码设置布局动画：
    ILayoutAnimationController controller = ILayoutAnimationController.generateController(AnimationUtils.loadAnimation(this,R.anim.activity_open_enter),0.8f,ILayoutAnimationController.IndexAlgorithm.INDEXSIMPLEPENDULUM);
    ll.setLayoutAnimation(controller);

    //一行代码直接搞定：
    ILayoutAnimationController.setLayoutAnimation(ll,R.anim.activity_open_enter,0.8f,ILayoutAnimationController.IndexAlgorithm.INDEXSIMPLEPENDULUM);

**方法setLayoutAnimation中各参数介绍：**
    /**
     * 根据传入的动画资源ID、单个子View动画延时、子View动画执行顺序算法枚举值 创建一个新的CustomLayoutAnimationController实例，
     * 将此实例作为参数为viewGroup设置布局动画
     * @param viewGroup
     * @param animResId
     * @param delay
     * @param indexAlgorithm
     */
    public static void setLayoutAnimation(@NonNull ViewGroup viewGroup,@AnimRes int animResId, float delay,@Nullable final IndexAlgorithm indexAlgorithm){
        Animation animation = AnimationUtils.loadAnimation(viewGroup.getContext(),animResId);
        setLayoutAnimation(viewGroup,animation,delay,indexAlgorithm);
    }
    
**注意：**
使用ILayoutAnimationController获取的ILayoutAnimationController实例，调用setOrder(int order)方法无效！