---
layout:     post
title:      Android高级渲染Shader（上）——基本用法
subtitle:   高级渲染了解下？
date:       2017-06-01
author:     陈再峰
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - Shader
---

在安卓中需要做一些渲染的UI的渐变效果。实现这些效果我们需要了解安卓渐变的使用。因此我们需要了解一个非常重要的类——Shader。

![Shader](http://upload-images.jianshu.io/upload_images/1930161-5b0dc864022239e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有五个类继承了Shader：

**BitmapShader：**位图图像渲染。
**LinearGradient：**线性渲染。
**SweepGradient：**渐变渲染/梯度渲染。
**RadialGradient：**环形渲染。
**ComposeShader：**组合渲染

####1、BitmapShader：位图图像渲染####

BitmapShader只作用于Bitmap，用Bitmap对绘制的图形进行渲染着色。
构造函数中需要传入图片的拉升模式TileMode，同时设置不同TileMode会展示的效果也是BitmapShader的重点！
```
mBitmapShader = new BitmapShader(mBitmap, Shader.TileMode.MIRROR, Shader.TileMode.MIRROR);
```
使用时，首先paint.setShader()，然后用canvas.draw();
```
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPaint.setShader(mBitmapShader);
        canvas.drawRect(0, 0, 800, 800, mPaint);
    }
```
这里将一个张图片去填充边长为800的正方形，长宽采用一样的拉伸模式。
**CLAMP**—— 是拉伸最后一个像素铺满。

![CLAMP](http://upload-images.jianshu.io/upload_images/1930161-9f635adbcd48d1bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**MIRROR**——是横向纵向不足处不断翻转镜像平铺。
![MIRROR](http://upload-images.jianshu.io/upload_images/1930161-38a1b2000bcfdaa3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**REPEAT** ——类似电脑壁纸，横向纵向不足的重复放置。
![REPEAT](http://upload-images.jianshu.io/upload_images/1930161-2fb83cc00398713d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####2、LinearGradient：线性渲染####
```
    /** Create a shader that draws a linear gradient along a line.
        @param x0           The x-coordinate for the start of the gradient line
        @param y0           The y-coordinate for the start of the gradient line
        @param x1           The x-coordinate for the end of the gradient line
        @param y1           The y-coordinate for the end of the gradient line
        @param  colors      The colors to be distributed along the gradient line
        @param  positions   May be null. The relative positions [0..1] of
                            each corresponding color in the colors array. If this is null,
                            the the colors are distributed evenly along the gradient line.
        @param  tile        The Shader tiling mode
    */
public LinearGradient(float x0, float y0, float x1, float y1, int colors[], float positions[],
            TileMode tile)
```
构造函数中参数的意思：
x0：渲染起点的X坐标
y0：渲染起点的Y坐标
x1：渲染终点的X坐标
y1：渲染终点的Y坐标
colors：渲染的颜色集合。
positions：渲染颜色所占的比例，如果传null，则均匀渲染.
tile : 拉伸模式，和BitmaopShaper类似。
**其他构造函数这里不做讲解，传入的参数含义也比较好理解**
```
mLinearGradient = new LinearGradient(0,500,500,500,new int[]{Color.RED,Color.BLUE,Color.GRAY,Color.GREEN},null, Shader.TileMode.MIRROR);
```
```
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPaint.setShader(mLinearGradient);
        canvas.drawRect(0, 0, 800, 800, mPaint);
    }
```
效果图：。
![LinearGradient  ](http://upload-images.jianshu.io/upload_images/1930161-43042453693452e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####3、SweepGradient：梯度渲染####
```
/**
     * A subclass of Shader that draws a sweep gradient around a center point.
     *
     * @param cx       The x-coordinate of the center
     * @param cy       The y-coordinate of the center
     * @param colors   The colors to be distributed between around the center.
     *                 There must be at least 2 colors in the array.
     * @param positions May be NULL. The relative position of
     *                 each corresponding color in the colors array, beginning
     *                 with 0 and ending with 1.0. If the values are not
     *                 monotonic, the drawing may produce unexpected results.
     *                 If positions is NULL, then the colors are automatically
     *                 spaced evenly.
     */
    public SweepGradient(float cx, float cy,
                         int colors[], float positions[]) 
```
构造函数中参数的意思：
cx：渲染圆形中心点的x坐标。
cy：渲染圆形中心点的y坐标。
colors ：渲染的颜色集合。
positions：渲染颜色所占的比例，如果传null，则均匀渲染。
**其他构造函数这里不做讲解，传入的参数含义也比较好理解**
```
mSweepGradient = new SweepGradient(250, 250, new int[]{Color.GREEN, Color.YELLOW, Color.RED}, null);
```
使用时，首先paint.setShader()，然后用canvas.draw();
```
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPaint.setShader(mSweepGradient);
        canvas.drawCircle(250, 250, 250, mPaint);
    }
```
效果图：

![SweepGradient](http://upload-images.jianshu.io/upload_images/1930161-dd538e11823a481f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


####4、RadialGradient：环形渲染####
```
/** Create a shader that draws a radial gradient given the center and radius.
        @param centerX  The x-coordinate of the center of the radius
        @param centerY  The y-coordinate of the center of the radius
        @param radius   Must be positive. The radius of the circle for this gradient.
        @param colors   The colors to be distributed between the center and edge of the circle
        @param stops    May be <code>null</code>. Valid values are between <code>0.0f</code> and
                        <code>1.0f</code>. The relative position of each corresponding color in
                        the colors array. If <code>null</code>, colors are distributed evenly
                        between the center and edge of the circle.
        @param tileMode The Shader tiling mode
    */
    public RadialGradient(float centerX, float centerY, float radius,
               @NonNull int colors[], @Nullable float stops[], @NonNull TileMode tileMode)
```
centerX：渲染圆形中心点的x坐标。
centerY：渲染圆形中心点的y坐标。
radius:渲染圆形的半径。
colors ：渲染的颜色集合。
stops：渲染颜色所占的比例，如果传null，则均匀渲染。
tileMode ：拉伸模式，和BitmaopShaper类似。
**其他构造函数这里不做讲解，传入的参数含义也比较好理解**
```
mRadialGradient = new RadialGradient(250, 250, 250, new int[]{Color.RED, Color.GREEN, Color.BLACK}, null, Shader.TileMode.CLAMP);
```
```
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPaint.setShader(mRadialGradient);
        canvas.drawCircle(250, 250, 250, mPaint);
    }
```
效果图：

![RadialGradient](http://upload-images.jianshu.io/upload_images/1930161-331cea159df9283e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####5、ComposeShader：组合渲染####
ComposeShader会将两种渲染叠加。为什么是两种呢？因为：
```
    /** Create a new compose shader, given shaders A, B, and a combining mode.
        When the mode is applied, it will be given the result from shader A as its
        "dst", and the result from shader B as its "src".
        @param shaderA  The colors from this shader are seen as the "dst" by the mode
        @param shaderB  The colors from this shader are seen as the "src" by the mode
        @param mode     The mode that combines the colors from the two shaders. If mode
                        is null, then SRC_OVER is assumed.
    */
    public ComposeShader(Shader shaderA, Shader shaderB, Xfermode mode) {
        mType = TYPE_XFERMODE;
        mShaderA = shaderA;
        mShaderB = shaderB;
        mXferMode = mode;
        init(nativeCreate1(shaderA.getNativeInstance(), shaderB.getNativeInstance(),
                (mode != null) ? mode.native_instance : 0));
    }

    /** Create a new compose shader, given shaders A, B, and a combining PorterDuff mode.
        When the mode is applied, it will be given the result from shader A as its
        "dst", and the result from shader B as its "src".
        @param shaderA  The colors from this shader are seen as the "dst" by the mode
        @param shaderB  The colors from this shader are seen as the "src" by the mode
        @param mode     The PorterDuff mode that combines the colors from the two shaders.
    */
    public ComposeShader(Shader shaderA, Shader shaderB, PorterDuff.Mode mode) {
        mType = TYPE_PORTERDUFFMODE;
        mShaderA = shaderA;
        mShaderB = shaderB;
        mPorterDuffMode = mode;
        init(nativeCreate2(shaderA.getNativeInstance(), shaderB.getNativeInstance(),
                mode.nativeInt));
    }
```
它的构造函数只能传两种渲染效果。（但是，是否可以在ComposeShader的构造函数中传入ComposeShader达到多种渲染效果叠加？）
构造函数中的第三个参数是设置叠加模式：http://blog.csdn.net/t12x3456/article/details/10432935
```
public class ComposeShaderTestView extends View {

    private ComposeShader composeShader;

    //位图渲染
    private BitmapShader mBitmapShader;
    private Bitmap mBitmap;

    //线性渲染
    private LinearGradient mLinearGradient;

    private Paint mPaint;

    public ComposeShaderTestView(Context context) {
        this(context, null);
    }

    public ComposeShaderTestView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public ComposeShaderTestView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.hy2);
        mBitmapShader = new BitmapShader(mBitmap, Shader.TileMode.REPEAT, Shader.TileMode.REPEAT);

        mLinearGradient = new LinearGradient(0, mBitmap.getHeight(), mBitmap.getWidth(), mBitmap.getHeight(), new int[]{Color.RED, Color.BLUE, Color.GRAY, Color.GREEN}, null, Shader.TileMode.MIRROR);

        //组合渲染
        composeShader = new ComposeShader(mBitmapShader, mLinearGradient, PorterDuff.Mode.MULTIPLY);

        mPaint = new Paint();
        mPaint.setAntiAlias(true);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPaint.setShader(composeShader);
        canvas.drawRect(0, 0, mBitmap.getWidth(), mBitmap.getHeight(), mPaint);
    }
}
```
这里将线性渲染和位图渲染叠加了，效果图：

![ComposeShader](http://upload-images.jianshu.io/upload_images/1930161-4864efb50dda4d53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果基本了解了Shader的用法，那就看一下看一下Shader的实例效果：
https://www.jianshu.com/p/3ded93e3b863

代码地址：https://github.com/AxeChen/Gradient
