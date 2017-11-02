### 浅谈Android系统中drawable的使用
>在Android系统中有很多有drawable相关的概念。比如BitmapDrawable、LayerDrawable、ScaleDrawable等。同时android系统中同样存在drawable-hdip、drawable-ldip等。在Android Studio中同样也存在mipmap-hdpi、mipmap-mdpi等。


##### 一、Android中的单位
**inch**

inch即为英寸，它表示设备的物理屏幕的对角线长度。 
比如手机的屏幕尺寸为5英寸，表示的就是手机的右上角与左下角之间的距离，其中1 inch = 2.54 cm

**px**

pixel简称为px，它表示屏幕的像素，也就是大家常说的屏幕分辨率。 
比如手机的分辨率为1920*1080，它表示屏幕的X方向上有1080个像素，Y方向上有1920个像素。

**pt**

pt类似于px，但常用于字体的单位。

**dpi和densityDpi**

dot per inch简称为dpi，它表示每英寸上的像素点个数，所以它也常为屏幕像素密度。 
在Android中使用DisplayMetrics中的densityDpi字段表示该值，并且不少文档中常用dpi来简化或者指代densityDpi。

**dp**

density-independent pixel简称为dip或者dp,它表示与密度无关的像素。
如果使用dp作为长度单位，那么该长度在不同密度的屏幕中显示的比例将保持一致。

##### 二、Android中drawable的区别
Android系统中为我们提供了不同分辨率的资源路径来让我们完成屏幕的适配。比如drawable-ldpi、drawable-hdpi。那么这些文件夹的区别在哪里呢？如果将同一张图片放到不同的文件夹，所占的内容又有何区别呢？

- drawable-ldpi：低像素密度（mipmap-ldpi）
- drawable-mdpi：中像素密度（mipmap-mdpi）
- drawable-hdpi：高像素密度（mipmap-hdpi）

通过上面的基础概念，我们可以推断，当资源放置的资源目录dpi越大，图片在内容中尺寸会越小。因为根据dpi像素密度的概念（px/dpi）。图片在内存中加载的尺寸越小，所占的内存就越小。
所以同一张图片在内存中占用的大小：ldpi>mdpi>hdpi。

下面通过源码的角度来观察下，我们知道在Drawable中的资源我们通过BitmapFactory.decodeResource工具类类生成Bitmap对象。
```java
public static Bitmap decodeResource(Resources res, int id, Options opts) {
    Bitmap bm = null;
    InputStream is = null; 

    try {
        final TypedValue value = new TypedValue();
        is = res.openRawResource(id, value);

        bm = decodeResourceStream(res, value, is, null, opts);
    } catch (Exception e) {
        /*  do nothing.
            If the exception happened on open, bm will be null.
            If it happened on close, bm is still valid.
        */
    } finally {
        try {
            if (is != null) is.close();
        } catch (IOException e) {
            // Ignore
        }
    }

    if (bm == null && opts != null && opts.inBitmap != null) {
        throw new IllegalArgumentException("Problem decoding into existing bitmap");
    }

    return bm;
}
```
在这里面，我们可以看到decodeResource方法的本质调用了decodeResourceStream方法，这个方法里面又有哪些内容呢？

```java
/**
 * Decode a new Bitmap from an InputStream. This InputStream was obtained from
 * resources, which we pass to be able to scale the bitmap accordingly.
 */
public static Bitmap decodeResourceStream(Resources res, TypedValue value,
        InputStream is, Rect pad, Options opts) {

    if (opts == null) {
        opts = new Options();
    }

    if (opts.inDensity == 0 && value != null) {
        final int density = value.density;
        if (density == TypedValue.DENSITY_DEFAULT) {
            opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;
        } else if (density != TypedValue.DENSITY_NONE) {
            opts.inDensity = density;
        }
    }

    if (opts.inTargetDensity == 0 && res != null) {
        opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
    }

    return decodeStream(is, pad, opts);
}
```
在这里我们可以看到系统创建了一个Options对象。这个对象可用于我们缩放Bitmap。然后判断inDesity密度，这里给的默认值DisplayMetrics.DENSITY_DEFAULT=160（即对应mdpi）;下面在
```java
if (opts.inTargetDensity == 0 && res != null) {
    opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
}
```
判断系统字眼然后赋值系统对应的像素密度，这样就形成了我们相同的图片在不同的文件夹中占用内存大小不同的情况。