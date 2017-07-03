## Android绘制验证码

>在前面[仿华为加载动画](http://blog.csdn.net/mr_dsw/article/details/73658213)、[仿网易音乐听歌识曲-麦克风动画](http://blog.csdn.net/mr_dsw/article/details/73730601)中，我们通过绘图的基础知识完成了简单的绘制。在本例中，我们将绘制常见的验证码。

### 一、效果图

![identify](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Android%E7%BB%98%E5%88%B6%E9%AA%8C%E8%AF%81%E7%A0%81/identifycode.png)


### 二、知识点与思路分析
通过上面的效果图观察，我们可以看到里面有绘制的随机线条，随机绘制的验证码。

- 绘制线条，直线或曲线
- 绘制文本，生成的验证码文本的绘制
- 绘制圆点。

### 三、代码编写

```java
/**
 * Created by Iflytek_dsw on 2017/7/3.
 */

public class IdentifyCodeUtil {
    private static final int CODE_NUMBER = 4;
    private static final int LINE_NUMBER = 5;
    private static final int POINT_NUMBER = 10;
    private StringBuffer stringBuffer = null;
    private Random random = new Random();
    //随机数数组
    private static final char[] CHARS = {
            '2', '3', '4', '5', '6', '7', '8', '9',
            'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'j', 'k', 'm',
            'n', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z',
            'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M',
            'N', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'
    };

    private static IdentifyCodeUtil instance;

    public static IdentifyCodeUtil getInstance(){
        if(instance == null){
            instance = new IdentifyCodeUtil();
        }
        return instance;
    }


    public Bitmap createBitmapCode(int width, int height){
        Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        canvas.drawColor(Color.LTGRAY);
        drawCodeText(canvas, width, height);
        drawLines(canvas, width, height);
        drawPoint(canvas, width, height);
        return bitmap;
    }

    /**
     * 返回验证码
     * @return  验证码生成的字符串
     */
    public String getIdentifyCode(){
        if(stringBuffer == null){
            return "";
        }
        return stringBuffer.toString();
    }

    /**
     * 生成验证码
     * @return
     */
    private String buildIdentifyCode(){
        StringBuffer stringBuffer = new StringBuffer();
        for(int i=0; i < CODE_NUMBER;i++){
            stringBuffer.append(CHARS[random.nextInt(CHARS.length)]);
        }
        Log.d("Code",stringBuffer.toString());
        return stringBuffer.toString();
    }

    /**
     * 绘制文本
     * @param canvas    画布
     * @param width     宽度
     * @param height    高度
     */
    private void drawCodeText(Canvas canvas,int width, int height){
        Paint paint = new Paint();
        paint.setTextSize(50);
        /**构建验证码code*/
        String text = buildIdentifyCode();
        float textLength = paint.measureText(text);
        int startMaxLength = (int) ((width - textLength) / 2);
        /**随机计算验证码绘制每次开头的位置*/
        int startPosition = random.nextInt(startMaxLength);
        //绘制文字
        for(int index = 0; index < text.length(); index++){
            /**生成旋转的角度*/
            int offsetDegree = random.nextInt(15);
            /**这里只会产生0和1，如果是1那么正旋转正角度，否则旋转负角度*/
            offsetDegree = random.nextInt(2) == 1 ? offsetDegree : -offsetDegree;
            canvas.save();
            //设置旋转
            canvas.rotate(offsetDegree, width / 2, height / 2);
            /**生成随机的颜色*/
            paint.setARGB(255, random.nextInt(200) + 20, random.nextInt(200) + 20,
                    random.nextInt(200) + 20);
            char tempChar = text.charAt(index);
            //给画笔设置随机颜色
            canvas.drawText(String.valueOf(tempChar), startPosition +index * textLength / text.length() +15,
                    height * 3 / 5f,paint);
            canvas.restore();
        }
    }

    /**
     * 生成干扰线
     * @param canvas
     * @param width
     * @param height
     */
    private void drawLines(Canvas canvas,int width, int height){
        Paint paint = new Paint();
        paint.setStrokeWidth(3);
        for(int i = 0;i < LINE_NUMBER;i++){
            paint.setARGB(255, random.nextInt(200) + 30, random.nextInt(200) + 30, random.nextInt(200) + 30);
            int startX = random.nextInt(width);
            int startY = random.nextInt(height);
            int endX = random.nextInt(width);
            int endY = random.nextInt(height);
            canvas.drawLine(startX, startY, endX, endY, paint);
        }
    }

    /**
     * 生成干扰点
     */
    private void drawPoint(Canvas canvas, int width, int height) {
        Paint paint = new Paint();
        paint.setStrokeWidth(3);
        paint.setColor(Color.GRAY);
        for(int i=0; i< POINT_NUMBER; i++){
            PointF pointF = new PointF(random.nextInt(width) + 10, random.nextInt(height) + 10);
            canvas.drawPoint(pointF.x, pointF.y, paint);
        }
    }
}
```
![identifycode](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%A7%86%E5%9B%BE%E5%9F%BA%E7%A1%80/%E5%9B%BE%E5%BA%93/Android%E7%BB%98%E5%88%B6%E9%AA%8C%E8%AF%81%E7%A0%81/identifycode.gif)
