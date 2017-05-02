## Android绘制工具Canvas

在Android自定义View的学习中，我们经常需要绘制，Canvas类就承担起绘制的作用。在Android中，绘制一个View需要四个基本的步骤：
1. 一个视图或者像素的承载体：Bitmap
2. 一个绘制方法的承载体：Canvas
3. 绘制物：Rect、Path、text、Bitmap
4. 绘制方式的承载体：Paint

#### 如何构建一个Canvas对象
通过查看Canvas的api得知：
- Canvas()：创建一个空的栅格画布。
- Canvas(Bitmap bitmap)：构建一个指定bitmap的画布。

构造方法有两种，一种是创建空的栅格画布画布对象用来绘制。另一种是构建一个Canvas对象，绘制在Bitmap上。

#### Canvas方法介绍
##### 1、clipXXX：裁剪画布
- clipPath(Path path)：裁剪掉指定的path区域的Canvas
- clipPath(Path path, Region.Op op)：裁剪掉指定的path区域的Canvas，同时指定与上次裁剪的类型。
- clipRect(int left, int top, int right, int bottom)：裁剪掉指定矩形的Canvas区域
- clipRect(float left, float top, float right, float bottom)：裁剪指定区域
- clipRect(RectF rect)：裁剪指定矩形区域
- clipRect(float left, float top, float right, float bottom, Region.Op op)
- clipRect(Rect rect)
- clipRegion(Region region)
- clipRegion(Region region, Region.Op op)

在ClipXX方法中，可以分为三类：Path类、Rect类、Region类。注意，**在ClipXX的方法中，通过ClipXX方法，即改变显示区域，但是坐标区域并没有改变。**在ClipXX方法中，我们需要重要介绍下Region.Op（区域操作符）的使用。

通过查看源码，Region.Op是一个枚举类型：
```java
// the native values for these must match up with the enum in SkRegion.h
public enum Op {
    DIFFERENCE(0),
    INTERSECT(1),
    UNION(2),
    XOR(3),
    REVERSE_DIFFERENCE(4),
    REPLACE(5);

    Op(int nativeInt) {
        this.nativeInt = nativeInt;
    }

    /**
     * @hide
     */
    public final int nativeInt;
}
```
第一裁剪区域A，第二裁剪区域B。介绍如下：
1. Region.Op.DIFFERENCE：A-B的差集，即裁剪Canvas区域显示A与B的差集。
2. Region.Op.INTERSECT：A∩B的交集，即裁剪Canvas区域显示A与B的差集。
3. Region.Op.UNION：A∪B的并集，即裁剪Canvas区域显示A与B的并集。
4. Region.Op.XOR：a⊕b的异或，即裁剪Canvas区域显示A与B的异或。如果A与B有相交，则A、B的并集减去相交的部分。如果A与B不想交，则显示A、B的并集。
5. Region.Op.REVERSE_DIFFERENCE：反转差集，即B-A生成的差集。即裁剪区域显示B-A的差集。
6. Region.Op.REPLACE：用当前要剪切的区域代替之前剪切过的区域。

在这里，通过简单的实例看下效果。
```java
private void createBitmap(){
    //创建一个空白的Bitmap对象
    Bitmap bitmap = Bitmap.createBitmap(400,400, Bitmap.Config.ARGB_8888);
    //绘制一个矩形区域
    Canvas canvas = new Canvas(bitmap);
    Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
    paint.setStyle(Paint.Style.FILL_AND_STROKE);
    paint.setColor(Color.parseColor("#ffaacc"));
    canvas.drawRect(0, 0, 500, 500,paint);

    //裁剪出一个矩形,区域A
    canvas.clipRect(100,100,300,300);

    //再次裁剪一个区域，区域B
    Path path = new Path();
    path.addCircle(100,100,100, Path.Direction.CW);
    //此处修改op的值，来进行判断。
    canvas.clipPath(path, Region.Op.REPLACE);

    //裁剪后，绘制Canvas的画布
    paint.setColor(Color.parseColor("#00aacc"));
    canvas.drawRect(0, 0, 500, 500,paint);
    iv_image.setImageBitmap(bitmap);
}
```
通过上面的简单案例，我们通过修改以下代码片段，来修改效果：
```java
//此处修改op的值，来进行判断。
canvas.clipPath(path, Region.Op.REPLACE);
```
**1、Region.Op.DIFFERENCE效果**

![IDFFERENCE](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Region.Op.DIFFERENCE.png)

**2、Region.Op.INTERSECT**

![INTERSECT](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Region.Op.INTERSECT.png)

**3、Region.Op.UNION**

![UNION](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Region.Op.UNION.png)

**4、Region.Op.XOR**

![XOR](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Region.Op.XOR.png)

**5、Region.Op.REVERSE_DIFFERENCE**

![REVERSE_DIFFERENCE](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Region.Op.REVERSE_DIFFERENCE.png)

**6、Region.Op.REPLACE**

![REPLACE](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Region.Op.REPLACE.png)

##### 2、drawXXX：绘制系列
