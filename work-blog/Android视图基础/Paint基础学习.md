## Android绘制机制Paint基础学习
顾名思义，画笔的作用就是用来设置我们绘制图形、文本、位图的样式和颜色等信息。

### 一、Paint的6个内部类

1. Paint.Align：设置画笔的对齐方式
2. Paint.Cap：设置画笔绘制Line和Path的起始描边样式。
3. Paint.FontMetrics：文字测量
4. Paint.FontMetricsInt：文字测量
5. Paint.Join：
6. Paint.Style：画笔的样式

#### 1、Paint.Align 设置画笔绘制文本的对齐方式。有三种对齐方式：

- Paint.Align.CENTER：居中
- Paint.Align.LEFT  ：左对齐
- Paint.Align.RIGHT：右对齐

通过下面简单的代码体验下：

```java
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    Paint paint = new Paint();
    paint.setColor(Color.RED);
    paint.setStyle(Paint.Style.STROKE);
    paint.setStrokeWidth(2);
    paint.setTextSize(20f);

    Path path = new Path();
    path.moveTo(100, 50);
    path.lineTo(100,150);
    canvas.drawPath(path, paint);

    paint.setColor(Color.GREEN);
    //居中
    paint.setTextAlign(Paint.Align.CENTER);
    canvas.drawText("I Love Android:CENTER", 100, 50, paint);
    //左对齐
    paint.setTextAlign(Paint.Align.LEFT);
    canvas.drawText("I Love Android:LEFT", 100, 100, paint);
    //右对齐
    paint.setTextAlign(Paint.Align.RIGHT);
    canvas.drawText("I Love Android:RIGHT", 100, 150, paint);

}
```

![Align](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Alian.png)

**通过上面可以看到，以绘制起点作为对齐轴线进行对称。**

#### 2、Paint.Cap 设置画笔针对Line、Path起始拐点的描边样式。有三种类型：

- BUTT：默认值，绘制的描边在预设值范围内。
- ROUND：圆滑型，起始位置会以圆形超出预设值。
- SQUARE：方形，起始位置以方形超出预设值。

```java
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    Paint paint = new Paint();
    paint.setColor(Color.RED);
    paint.setStyle(Paint.Style.STROKE);
    paint.setStrokeWidth(2);
    canvas.drawLine(100, 50, 100, 350,paint);
    canvas.drawLine(300, 50, 300, 350,paint);

    paint.setStrokeWidth(20);
    paint.setColor(Color.GREEN);
    paint.setStrokeCap(Paint.Cap.BUTT);
    canvas.drawLine(100, 100, 300, 100, paint);

    paint.setStrokeCap(Paint.Cap.ROUND);
    canvas.drawLine(100, 200, 300, 200, paint);

    paint.setStrokeCap(Paint.Cap.SQUARE);
    canvas.drawLine(100, 300, 300, 300, paint);
}
```
![Cap](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Cap.png)

#### 3、Paint.Join 设置画笔针对Line、Curve和Path之间的连接样式，有三种类型：

- Paint.Join.MITER：默认值，直线的方式连接
- Paint.Join.ROUND：圆角的连接方式
- Paint.Join.BEVEL：斜线的连接方式

```java
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    Paint paint = new Paint();
    paint.setColor(Color.RED);
    paint.setStyle(Paint.Style.STROKE);
    paint.setStrokeWidth(2);
    canvas.drawLine(100, 50, 100, 350,paint);
    canvas.drawLine(300, 50, 300, 350,paint);
    paint.setStrokeWidth(20);
    paint.setColor(Color.GREEN);

    Path path = new Path();
    path.moveTo(100, 100);
    path.lineTo(300, 100);
    path.rLineTo(0,30);
    canvas.save();
    for(int i=0;i<3;i++){
        switch (i){
            case 0:
                paint.setStrokeJoin(Paint.Join.MITER);
                canvas.drawPath(path, paint);
                break;
            case 1:
                canvas.translate(0, 60);
                paint.setStrokeJoin(Paint.Join.ROUND);
                canvas.drawPath(path, paint);
                break;
            case 2:
                canvas.translate(0, 60);
                paint.setStrokeJoin(Paint.Join.BEVEL);
                canvas.drawPath(path, paint);
                break;
        }
    }
    canvas.restore();
}
```

![join](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Join.png)

#### 4、Paint.Style设置画笔的样式，有三种类型：

- Paint.Style.STROKE：描边
- Paint.Style.FILL：填充
- Paint.Style.FILL_AND_STROKE：描边+填充

```java
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    Paint paint = new Paint();
    paint.setColor(Color.RED);
    paint.setStrokeWidth(2);
    canvas.save();
    for(int i=0;i<3;i++){
        switch (i){
            case 0:
                paint.setStyle(Paint.Style.STROKE);
                canvas.drawRect(100, 100, 200, 200, paint);
                break;
            case 1:
                canvas.translate(0, 160);
                paint.setStyle(Paint.Style.FILL);
                canvas.drawRect(100, 100, 200, 200, paint);
                break;
            case 2:
                canvas.translate(0, 160);
                paint.setStyle(Paint.Style.FILL_AND_STROKE);
                canvas.drawRect(100, 100, 200, 200, paint);
                break;
        }
    }
    canvas.restore();
}
```
![stroke](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/stroke.png)

通过上面我们看到Fill和Fill_AND_STROKE的区别不是很明显。但是通过对属性注释的了解，**如果设置Paint的属性为Fill，则会忽略Paint其它与Stroke相关的属性，比如setStrokeCap、etStrokeJoin、setStrokeWidth，这些属性都依赖于Stroke属性。**


#### 5、Paint.FontMetrics字体测量
注：本部分讲解，取自[爱哥博客](http://blog.csdn.net/aigestudio/article/details/41447349)

```java
/**
 * Class that describes the various metrics for a font at a given text size.
 * Remember, Y values increase going down, so those values will be positive,
 * and values that measure distances going up will be negative. This class
 * is returned by getFontMetrics().
 */
public static class FontMetrics {
    /**
     * The maximum distance above the baseline for the tallest glyph in
     * the font at a given text size.
     */
    public float   top;
    /**
     * The recommended distance above the baseline for singled spaced text.
     */
    public float   ascent;
    /**
     * The recommended distance below the baseline for singled spaced text.
     */
    public float   descent;
    /**
     * The maximum distance below the baseline for the lowest glyph in
     * the font at a given text size.
     */
    public float   bottom;
    /**
     * The recommended additional space to add between lines of text.
     */
    public float   leading;
}
```
![fontMetrics](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/FontMetrics.gif)

这张图很简单但是也很扼要的说明了top,ascent,descent,bottom,leading这五个参数。首先我们要知道Baseline基线，在Android中，文字的绘制都是从Baseline处开始的，Baseline往上至字符最高处的距离我们称之为ascent（上坡度），Baseline往下至字符最底处的距离我们称之为descent（下坡度），而leading（行间距）则表示上一行字符的descent到该行字符的ascent之间的距离，top和bottom文档描述地很模糊，其实这里我们可以借鉴一下TextView对文本的绘制，TextView在绘制文本的时候总会在文本的最外层留出一些内边距，为什么要这样做？因为TextView在绘制文本的时候考虑到了类似读音符号，可能大家很久没写过拼音了已经忘了什么叫读音符号了吧……下图中的A上面的符号就是一个拉丁文的类似读音符号的东西：

![Font](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Font.png)

