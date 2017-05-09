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

##### 2、drawXXX系列方法
Canvas是我们的画布，给我们提供了一系列的方法满足我们在画布上进行绘制的需求，通过对api的分类，主要可以分为以下几种的绘制：

- drawArc：绘制圆弧。
- drawARGB：给整个可见的画布区域绘制颜色，即背景色。
- drawBitmap：绘制Bitmap对象。
- drawBitmapMesh：
- drawCircle：绘制圆形。
- drawColor：给整个可见的画布区域绘制颜色，即背景色。同drawARGB。
- drawLine(s)：绘制线条。
- drawOval：绘制椭圆。
- drawPaint：给Canvas设置画笔paint。
- drawPatch：绘制.9.png图片。
- drawPath：绘制Path路径。
- drawPicture：绘制Picture对象。
- drawPoint(s)：绘制点。
- drawRect：绘制矩形。
- drawRoundRect：绘制圆角矩形。
- drawRGB：给整个可见的画布区域绘制颜色，即背景色。同drawARGB。
- drawText：绘制文本。

**1、drawArc：绘制圆弧**

- drawArc(RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint)
- public void drawArc(float left, float top, float right, float bottom, float startAngle,float sweepAngle, boolean useCenter, Paint paint)

以上两个方法中，本质的核心都是根据指定的RectF矩形约束的范围进行圆弧的绘制。重点的以下几个参数的约束。

- startAngle：起始角度，这里需要注意：如果起始角度<0或>=360度，则取angle %360作为起始角度。
- sweepAngle：如果旋转角度>360度，则绘制全部。这点不同于path.arcTo的angle%360度取余数。如果angle是负数，则进行取余处理。
- useCenter：如果设置中心，则绘制圆弧的时候连接起点、中点和终点的连线闭合。

>此处补充一点，负数的余数是负整数<=0，正数的余数是正整数>=0。绘制的圆弧会自动进行缩放来填充指定矩形的椭圆形状，不会超过指定的矩形区域大小。换种说法，绘制的圆弧片段是矩形的内切椭圆上的指定角度的圆弧。

```java
/**
 * 绘制圆弧
 */
private void drawArc(){
    //创建空白的Bitmap对象
    Bitmap bitmap = Bitmap.createBitmap(500, 500, Bitmap.Config.ARGB_8888);
    Canvas canvas = new Canvas(bitmap);
    Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
    paint.setColor(Color.BLUE);
    paint.setStyle(Paint.Style.STROKE);
    //绘制我们约束的矩形的范围，以便形成对比
    RectF rect = new RectF(100,200,400,400);
    canvas.drawRect(rect,paint);

    //绘制圆弧的形状
    paint.setColor(Color.CYAN);
    canvas.drawArc(rect,-180,90,false,paint);
    canvas.drawArc(rect,20,80,true,paint);
    iv_image.setImageBitmap(bitmap);
}
```
效果图：

**2、drawBitmap系列：绘制Bitmap资源**
- drawBitmap(int[] colors, int offset, int stride, float x, float y, int width, int height, boolean hasAlpha, Paint paint)：将集合中的Colors绘制成Bitmap对象。需要开启硬件加速器，已经在api21中过期。
- drawBitmap(Bitmap bitmap, Matrix matrix, Paint paint)：绘制一个给定matrix的Bitmap对象。
- drawBitmap(int[] colors, int offset, int stride, int x, int y, int width, int height, boolean hasAlpha, Paint paint)：将集合中的Colors绘制成Bitmap对象。需要开启硬件加速器，已经在api21中过期。
- drawBitmap(Bitmap bitmap, Rect src, RectF dst, Paint paint)：绘制一个适应给定RectF大小的Bitmap资源。
- drawBitmap(Bitmap bitmap, float left, float top, Paint paint)：在left、top的位置出绘制一个Bitmap资源。
- drawBitmap(Bitmap bitmap, Rect src, Rect dst, Paint paint)

绘制Bitmap对象。
```java
private class BitmapView extends View{
    Paint paint = null;
    Bitmap bitmap = null;
    public BitmapView(Context context) {
        super(context);
        paint = new Paint(Paint.ANTI_ALIAS_FLAG);
        paint.setFlags(Paint.DITHER_FLAG);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        bitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher)
                .copy(Bitmap.Config.ARGB_8888,true);
        drawBitmap(canvas, 100, 100, paint);
        drawBitmapWithMatrix(canvas);
        drawBitmapWithRect(canvas);
        drawBitmapColors(canvas);
    }

    /**
     * 在指定的位置绘制Bitmap对象
     * @param canvas
     * @param left
     * @param top
     * @param paint
     */
    private void drawBitmap(Canvas canvas, int left, int top, Paint paint){
        canvas.drawBitmap(bitmap, left, top,paint);
    }

    /**
     * 绘制指定Matrix变换的bitmap
     * @param canvas
     */
    private void drawBitmapWithMatrix(Canvas canvas){
        Matrix matrix = new Matrix();
        matrix.setTranslate(200,200);
        matrix.postRotate(30);
        canvas.drawBitmap(bitmap, matrix, paint);
    }

    /**
     * 绘制指定DstRect大小的Bitmap
     * @param canvas
     */
    private void drawBitmapWithRect(Canvas canvas){
        Rect rectSrc = new Rect(0,0,100,100);
        Rect rectDst = new Rect(100,100,300,400);
        canvas.drawBitmap(bitmap, rectSrc,rectDst,paint);
    }

    private void drawBitmapColors(Canvas canvas){
        Bitmap bitmap = Bitmap.createBitmap(200, 200, Bitmap.Config.ARGB_8888);
        canvas.setBitmap(bitmap);
        canvas.drawBitmap(new int[]{Color.RED,Color.BLUE,Color.GREEN},
                1,1,0,0,200,200,false,paint);
    }
}
```



**3、drawColor系列**
Canvas为我们提供了绘制背景色的方法。

- drawARGB(int a, int r, int g, int b)
- drawColor(int color)
- drawColor(int color, PorterDuff.Mode mode)
- drawRGB(int r, int g, int b)

首先简单介绍下几个参数：
- a：透明度，取值(0..255)，0表示完全透明，255表示完全不透明
- r：三元素中的红，取值(0..255)
- g：三元素中的绿，取值(0..255)
- b：三元素中的蓝，取值(0..255)
- PorterDuff.Mode：颜色混合模式，共有18中混合方式。

在drawColor系列的方法中，我认为掌握的难点就是针对PorterDuff.Mode的掌握和使用。

1. PorterDuff.Mode.CLEAR：所绘制不会提交到画布上。
2. PorterDuff.Mode.SRC：显示上层绘制图片
3. PorterDuff.Mode.DST：显示下层绘制图片
4. PorterDuff.Mode.SRC_OVER：正常绘制显示，上下层绘制叠盖。
5. PorterDuff.Mode.DST_OVER：上下层都显示。下层居上显示。
6. PorterDuff.Mode.SRC_IN：取两层绘制交集。显示上层。
7. PorterDuff.Mode.DST_IN：取两层绘制交集。显示下层。
8. PorterDuff.Mode.SRC_OUT：取上层绘制非交集部分。
9. PorterDuff.Mode.DST_OUT：取下层绘制非交集部分。
10. PorterDuff.Mode.SRC_ATOP：取下层非交集部分与上层交集部分
11. PorterDuff.Mode.DST_ATOP：取上层非交集部分与下层交集部分
12. PorterDuff.Mode.XOR： 异或：去除两图层交集部分
13. PorterDuff.Mode.DARKEN：取两图层全部区域，交集部分颜色加深
14. PorterDuff.Mode.LIGHTEN： 取两图层全部，点亮交集部分颜色
15. PorterDuff.Mode.MULTIPLY： 取两图层交集部分叠加后颜色
16. PorterDuff.Mode.SCREEN：取两图层全部区域，交集部分变为透明色
17. PorterDuff.Mode.ADD：
18. PorterDuff.Mode.OVERLAY：

>注意：在上面的描述中下层的图层对应的是Dst图层，上层对应的是Src图层。

通过一段测试代码进行测试该功能的使用。

```java
/**
 * PorterDuff的使用测试
 */
private void drawColorWithPorterDuff(){
    //首先我们创建一个绘制的背景画布
    Bitmap bitmapBG = Bitmap.createBitmap(400, 400, Bitmap.Config.ARGB_8888);
    Bitmap bitmapDst = createDst(200, 200);
    Bitmap bitmapSrc = createSrc(200, 200);
    Canvas canvas = new Canvas(bitmapBG);
    Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
    canvas.drawColor(Color.GRAY);

    int sc = canvas.saveLayer(0, 0, 400, 400,null,
            Canvas.CLIP_TO_LAYER_SAVE_FLAG);
    //创建Dst图,绘制出来
    canvas.drawBitmap(bitmapDst,0,0,paint);
    //创建Src图
    paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_OVER));
    canvas.drawBitmap(bitmapSrc,100,100,paint);
    canvas.restoreToCount(sc);

    iv_image.setImageBitmap(bitmapBG);
}
```
展示效果图：
![porterduff](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/PorterDuffMode.png)

**4、drawXXX绘制图形系列**
