---
tags:
    - Android
	- 自定义View
	- 高级UI
	- Shader
---



我们在Android的Shader（上）中已经知道Shader的基本用法了。接下来实现一些具体的效果。

Android高级渲染Shader（上）：http://www.jianshu.com/p/5fb82b189094

**BitmapShader：**实现圆形图像
**LinearGradient：**实现渐变文字
**SweepGradient：**实现雷达效果
**RadialGradient ：**水波纹点击效果


####Matrix：矩阵
在实现实际的效果前，首先了解一下Matrix。
Matrix是**矩阵**，调用矩阵的一些方法我们可以对像素点进行平移，旋转，缩放等操作。矩阵在结合Shader的场合也比较多，使用起来比较方便。
```
Shader.setLocalMatrix(Matrix);
```
大概了解矩阵可以看这里：http://blog.csdn.net/cquwentao/article/details/51445269

####1、BitmapShader——实现圆形图像
实现这个效果非常简单，BitmapShader使用Bitmap对绘制的图形进行渲染，那我们只要用mPaint.setShader();之后再画一个圆就行了。
```
/**
 * @Author Chen
 * @Create 2017/5/31 0031
 */

public class RoundImageView extends View {

    private Bitmap mBitmap;
    private BitmapShader mBitmapShader;

    private Paint mPaint;

    private RectF rect;

    public RoundImageView(Context context) {
        this(context, null);
    }

    public RoundImageView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public RoundImageView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.hy2);
        mBitmapShader = new BitmapShader(mBitmap, Shader.TileMode.REPEAT, Shader.TileMode.REPEAT);

        mPaint = new Paint();
        mPaint.setAntiAlias(true);

        rect = new RectF();
        rect.left = 0;
        rect.top = mBitmap.getHeight();
        rect.bottom = mBitmap.getHeight() * 2;
        rect.right = mBitmap.getWidth();

    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPaint.setShader(mBitmapShader);
        canvas.drawCircle(mBitmap.getWidth() / 2, mBitmap.getWidth() / 2, mBitmap.getWidth() / 2, mPaint);

        canvas.drawOval(rect, mPaint);
    }
}

```
效果展示：


![圆形图案](http://upload-images.jianshu.io/upload_images/1930161-907bf0fc9637b7d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####2、LinearGradient——实现渐变文字
这里结合了Matrix的平移操作。
首先确定要渐变的宽度，然后通过Matrix的setTranslate移动这个渐变同时刷新界面。
```

/**
 * Created by Chen on 2017/5/21.
 */

public class GradientTextView extends android.support.v7.widget.AppCompatTextView {

    private LinearGradient mLinearGradient;
    private TextPaint mPaint;

    private Matrix mMatrix;

    private float mTranslate;
    private float DELTAX = 20;

    public GradientTextView(Context context) {
        this(context, null);
    }

    public GradientTextView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public GradientTextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mPaint = getPaint();
        mMatrix = new Matrix();

        String text = getText().toString();

        float textWith = mPaint.measureText(text);
        // 确定文字的宽度
        int gradientSize = (int) (textWith / text.length() * 3);

        mLinearGradient = new LinearGradient(0, 0, gradientSize, 0, new int[]{
                0x22ffffff, 0xffffffff, 0x22ffffff}, null, Shader.TileMode.CLAMP
        );

        mPaint.setShader(mLinearGradient);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        //实现轮播效果
        mTranslate += DELTAX;
        float textWidth = getWidth();
        if (mTranslate > textWidth -50 || mTranslate < 1) {
            DELTAX = -DELTAX;
        }
        //移动渐变
        mMatrix.setTranslate(mTranslate, 0);
        mLinearGradient.setLocalMatrix(mMatrix);
        postInvalidateDelayed(50);
    }
}


```
效果展示：
![渐变文字](http://upload-images.jianshu.io/upload_images/1930161-50bdbe3cdb4e8ab4.gif?imageMogr2/auto-orient/strip)

####3、SweepGradient——实现雷达效果
这里结合了Matrix的旋转操作。
先画出渐变的图形，然后通过matrix.setRotate来实现旋转同时刷新界面。
```
import com.mg.axe.gradient.ScreenUtils;

/**
 * @Author Chen
 * @Create 2017/5/31 0031
 */

public class RadarView extends View {

    private Paint mPaint;
    private SweepGradient mSweepGradient;

    private Matrix matrix;
    private int mWidth = 0;

    private RectF rectF;

    private int lineDistance;

    /**
     * 旋转的角度
     **/
    private int degree = 0;

    public RadarView(Context context) {
        this(context, null);
    }

    public RadarView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public RadarView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        mWidth = ScreenUtils.getScreenWidth(context)[0];

        lineDistance = mWidth / 2 / 2;

        mPaint = new Paint();
        mPaint.setAntiAlias(true);

        matrix = new Matrix();

        mSweepGradient = new SweepGradient(
                mWidth / 2, mWidth / 2, new int[]{Color.TRANSPARENT, Color.parseColor("#41CC00")}, null);

        rectF = new RectF();


    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.drawColor(Color.BLACK);

        for (int i = 0; i < 2; i++) {
            rectF.top = 10 + i * lineDistance;
            rectF.left = 10 + i * lineDistance;
            rectF.bottom = (mWidth - 10) - i * lineDistance;
            rectF.right = (mWidth - 10) - +i * lineDistance;

            //先画圆圈
            mPaint.setStyle(Paint.Style.STROKE);
            mPaint.setStrokeWidth(10);
            mPaint.setColor(Color.parseColor("#05E60B"));
            canvas.drawArc(rectF, 0, 360, false, mPaint);
        }

        //画点
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(15);
        canvas.drawPoint(mWidth / 2, mWidth / 2, mPaint);

        //画扫描框
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setShader(mSweepGradient);
        canvas.drawCircle(mWidth / 2, mWidth / 2, mWidth / 2 - 5, mPaint);

        // 一定要Reset（具体原因还在调研中）
        mPaint.reset();
        //使用Matrix旋转
        mSweepGradient.setLocalMatrix(matrix);
        matrix.setRotate(degree, mWidth / 2, mWidth / 2);
        degree++;
        if (degree > 360) {
            degree = 0;
        }
        postInvalidate();
    }
}
```
效果展示：

![雷达效果](http://upload-images.jianshu.io/upload_images/1930161-eb3be7e3b56605c7.gif?imageMogr2/auto-orient/strip)


####4、RadialGradient ——水波纹点击效果
这里的逻辑是：
DOWN事件：画一个小圆。
UP事件：小圆散开，当圆的半径扩大到最大值时消失。
最后消失的逻辑将半径设置为0即可。
```
/**
 * @Author Chen
 * @Create 2017/5/31 0031
 */

public class WaveView extends View {

    private Paint mPaint;
    private RadialGradient mRadialGradient;

    /**
     * 按下时的的大小
     */
    private static final int DEFAULT_FIRST_CLICK_RADIUS = 50;


    /**
     * 最大的半徑
     */
    private static final int DEFAULT_MOST_RADIUS = 400;


    private int mRadius = 0;

    /**
     * 按下去的X坐标
     */
    private int cX = 0;

    /**
     * 按下去时Y的坐标
     */
    private int cY = 0;

    /**
     * 是否触发UP
     */
    private boolean isUP = false;

    public WaveView(Context context) {
        this(context, null);
    }

    public WaveView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public WaveView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        mPaint = new Paint();
        mPaint.setAntiAlias(true);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.save();
        canvas.drawColor(Color.WHITE);
        canvas.drawCircle(cX,cY,mRadius,mPaint);
        if(isUP){
            mRadius+=20;
        }
        //渐变的圆圈是否到达最大值
        if(mRadius> DEFAULT_MOST_RADIUS){
            isUP = false;
            //设置mRadius为0圆圈就会消失了
            mRadius = 0;
        }
        invalidate();
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                cX = (int) event.getX();
                cY = (int) event.getY();
                //点击时画第一个小圆
                setGradient();
                isUP = false;
                break;
            case MotionEvent.ACTION_UP:
                isUP = true;
                break;
        }

        return true;
    }


    private void setGradient(){
        mRadius = DEFAULT_FIRST_CLICK_RADIUS;
        mRadialGradient = new RadialGradient(cX, cY, mRadius, new int[]{Color.parseColor("#F0F0F0"), Color.parseColor("#F0F0F0")}, null, Shader.TileMode.CLAMP);
        mPaint.setShader(mRadialGradient);
        invalidate();
    }


}

```
效果展示：

![点击水波纹](http://upload-images.jianshu.io/upload_images/1930161-7037f5cb1c32085f.gif?imageMogr2/auto-orient/strip

如果对基本用法还不熟悉，看这里：https://www.jianshu.com/p/3ded93e3b863https://www.jianshu.com/p/3ded93e3b863z代码地址：[https://github.com/AxeChen/Gradient](https://github.com/AxeChen/Gradient)
