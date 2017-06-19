## Android绘图基础Path、PathMeasure
前面我们队Canvas和Paint的基础有了一定的了解，针对Path，我以前也进行总结过[Android基础之Path类的使用](http://blog.csdn.net/mr_dsw/article/details/48931515)。现在在做进一步的整理，毕竟每个阶段理解的不同。

### 一、Path对应的三个内部类
- Path.Direction：Path路径绘制方向
- Path.FillType：Path对象的填充类型
- Path.Op：Path对象的相交类型

#### 1、Path.Direction：绘制方向，包含两个类型。

- CCW ：逆时针绘制
- CW ：顺时针绘制

注意，这里最明显的就是影响drawTextOnPath中text的绘制方向。

#### 2、Path.FillType

设置Path的填充类型。

- WINDING ：
- EVEN_ODD 
- INVERSE_WINDING 
- INVERSE_EVEN_ODD


#### 3、Path.Op

Op代表Path对象的裁剪类型。

- DIFFERENCE ：差集。A-B
- INTERSECT ：交集。A∩B
- REVERSE_DIFFERENCE ：反差集。B-A
- UNION ：并集。A∪B
- XOR ：异或。a⊕b的异或，即裁剪Canvas区域显示A与B的异或。如果A与B有相交，则A、B的并集减去相交的部分。如果A与B不想交，则显示A、B的并集。


### 二、Path方法简介

#### 1、add类方法
addXX类的方法可以完成在Path的对象中添加响应的图形结构。

- addArc(RectF oval, float startAngle, float sweepAngle)：圆弧形状
- addCircle(float x, float y, float radius, Path.Direction dir)：添加圆形。
- addOval(RectF oval, Path.Direction dir)：添加椭圆形状
- addPath(Path src, float dx, float dy)：添加Path路径
- addRect(float left, float top, float right, float bottom, Path.Direction dir)：添加矩形的形状
- addRoundRect(RectF rect, float[] radii, Path.Direction dir)：添加圆角矩形

#### 2、	moveTo(float x, float y)：移动Path绘制的起点位置。
比如默认情况下，Path绘制的起点位置是(0,0)，我们通过moveTo(100, 100)进行起点的移动。现在起点位置就是(100, 100)。

#### 3、lineTo(float x, float y)：从起点连接到点（x，y）的线。


#### 4、op裁剪类方法

- op(Path path1, Path path2, Path.Op op)：根据指定的类型裁剪Path
- op(Path path, Path.Op op)

#### 5、quadTo(float x1, float y1, float x2, float y2)：绘制二阶贝塞尔曲线。
绘制二阶的贝塞尔曲线。

![Besai](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/B%C3%A9zier_2_big.gif)


#### 6、	cubicTo(float x1, float y1, float x2, float y2, float x3, float y3)：绘制三阶贝塞尔曲线。


### 二、PathMeasure工具类
上面简要介绍了Path的基本使用，所以这次我们介绍Path的辅助工具类——PathMeasure。见名思意，我们可以通过PathMeasure工具类完成Path的长度以及对应点的位置获取。

#### 1、构造方法
- public PathMeasure() ：构造一个未绑定Path的PathMeasure对象。
- public PathMeasure(Path path, boolean forceClosed)：构造一个绑定Path的PathMeasure对象。

#### 2、public void setPath(Path path, boolean forceClosed)
添加一个Path对象

#### 3、public float getLength()
获取当前设定的Path路径的长度，如果没有设置Path，则长度为0

#### 4、public boolean getPosTan(float distance, float pos[], float tan[])
根据当前位置距离的起点位置长度，计算当前点的坐标。


