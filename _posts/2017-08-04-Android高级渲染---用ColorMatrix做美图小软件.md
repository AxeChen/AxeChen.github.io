---
tags:
    - Android
	- 自定义View
	- 高级UI
	- 滤镜
	- ColorMatrix
---


> 隔了一年多去看这篇文章，发现很多地方会有问题，比如7.0后拍照的问题，当时用的6.0的系统，所以7.0以上的系统一定会崩溃。还有就是加完滤镜之后无法修改模糊效果。
源码等可能暂时不维护，存在不少问题。

###1、效果展示
用ColorMatrix可以调节图片颜色比例，做到滤镜的效果。这里用ColorMatrix基本使用写了一个小的APP，源码地址：https://github.com/AxeChen/ColorFilter 
先上一波效果图：
![效果图一](http://upload-images.jianshu.io/upload_images/1930161-3d8c521b51dfa8be.gif?imageMogr2/auto-orient/strip)
![效果图二](http://upload-images.jianshu.io/upload_images/1930161-b5885bcbb6c83505.gif?imageMogr2/auto-orient/strip)
利用安卓自带的API即可完成一些简单的图片滤镜，调整图片的对比度亮度等属性。
###2、实现思路
**主要思路：**选择相册图片或者拍照获取图片，然后通过修改ColorMatrix来实现各种不同的滤镜效果或者通过修改BlurMaskFilter来实现边框的虚化等效果。
###3、主要技术
####3.1、主要技术点
（1）颜色矩阵ColorMatrix的应用（滤镜，调整亮度等主要用到的技术）。
（3）BlurMaskFilter边框虚化。
（4）6.0相机相册权限处理。
（5）Bitmap的一些处理：压缩，生成新的bitmap等。
（6）其他的一些技术点：一些控件的简单使用，例如（RecyclerView，Toobar）、分享图片等。

####3.2 颜色矩阵ColorMatrix
应用ColorMatrix可以调节图片颜色比例，实现一些滤镜，调节图片亮度、对比度等效果。
######3.2.1 ColorMatrix简单介绍
在Android中使用颜色矩阵ColorMatrix，来处理图像的色彩效果。对于图像的每个像素点，都有一个颜色矩阵A用来保存颜色的RGBA值。在处理图像是将颜色矩阵和颜色矩阵分量C相乘。

![颜色矩阵](http://upload-images.jianshu.io/upload_images/1930161-1f3b6fd8aa4d918f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![颜色处理](http://upload-images.jianshu.io/upload_images/1930161-4c60173c34959535.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



在安卓中使用一个一维数组表示颜色矩阵：
```
    public static final float[] src = {
            1f, 0, 0, 0, 0,    
            0, 1f, 0, 0, 0,
            0, 0, 1f, 0, 0,
            0, 0, 0, 1f, 0,
    };
```
如果对颜色矩阵基本知识有些难理解。可以不纠结这些基本的知识，只抓住重点：**只要改变颜色矩阵对应的数组中元素的值，就可以改变图片中颜色的比例！**

对应一维数组控制的颜色属性：
第一行决定新的颜色值中的红色R。
第二行决定新的颜色值中的绿色G。
第三行决定新的颜色值中的蓝色B。
第四行决定新的颜色值中的透明度A。
第五列表示每个颜色的偏移量。

**因此只要合理改变这个数组中不同元素的值就可以实现不同颜色效果了。**

######3.2.1 ColorMatrix使用
直接上关键代码：
```
public class FilterImageView extends FilterView {

     // ... 省略若干代码

    private Paint paint;
    private ColorMatrixColorFilter colorMatrixColorFilter;
    private ColorMatrix colorMatrix;


    @Override
    protected void onDraw(Canvas canvas) {
        paint.reset();
        paint.setAntiAlias(true);
        if (colorMatrixColorFilter != null) {
               paint.setColorFilter(colorMatrixColorFilter);
         }
        canvas.drawBitmap(bitmap, null, rectF, paint);
        canvas.save();
    }

    @Override
    public void setFloat(float[] floats) {
        colorMatrix = new ColorMatrix();
        colorMatrix.set(floats);
        colorMatrixColorFilter = new ColorMatrixColorFilter(colorMatrix);
        invalidate();
    }


   // ... 省略若干代码
}

```
代码步骤：
1、初始化ColorMatrixColorFilter 和 ColorMatrix 。
2、在onDraw()方法中通过paint.setColorFilter()方法将ColorMatrixColorFilter设置给paint。
3通过 canvas.drawBitmap()将Bitmap画出来即可。（不仅仅是drawBitmap只要drawXX方法中传入了设置过ColorMatrixColorFilter的paint都可实现颜色改变的效果）。

######3.2.2 修改颜色矩阵对应的数组
改变颜色矩阵对应的数组的数组元素值，可以展示不同的颜色效果。
**一些简单的修改：**
1、改变颜色的偏移量的值（下面的r、g、b、a）；
2、改变对应 RGBA（下面的R、G、B、A） 值的系数。
如下说明：
```
 public static final float[] src = {
            R, 0, 0, 0, r,          
            0, G, 0, 0, g,          
            0, 0, B, 0, b,          
            0, 0, 0, A, a,          
    };
```
以上：
R、G、B的值可以增大或者变小。
A的范围为0f-1f（可以设置比1f更大，但是效果和1f是一样的）
而偏移量r、g、b、a的范围是0f-255f（可以设置比255f更大，但是效果和255f是一样的）
改变R、G、B、A或者r、g、b、a的值都可以改变图片的颜色的比例和透明度。
这些值越大，图片中对应颜色所占的比例就越大或者越不透明。
例如：
```
    public static final float[] green = {
            1f, 0, 0, 0, 0,
            0, 1.2f, 0, 0, 0,
            0, 0, 1f, 0, 0,
            0, 0, 0, 1f, 0,
    };
```
以上把G值调整成1.2f 。就能增加图片中绿色的比例。

![绿色加强](http://upload-images.jianshu.io/upload_images/1930161-3256298ed836d739.gif?imageMogr2/auto-orient/strip)

**一些复杂的修改：**
通过调节数组中多个元素来做出不同的效果。
例如灰度效果：
```
    /**
     * 灰度效果
     */
    public static final float[] gray = {
            0.213f, 0.715f, 0.072f, 0, 0,
            0.213f, 0.715f, 0.072f, 0, 0,
            0.213f, 0.715f, 0.072f, 0, 0,
            0, 0, 0, 1, 0,
    };
```
![灰度效果](http://upload-images.jianshu.io/upload_images/1930161-e689f9e050c6d905.gif?imageMogr2/auto-orient/strip)
修改数组中多个元素的值来改变图片中颜色的比例，从而达到灰度的效果。

######3.2.2 调节亮度和对比度
ColorMatrix提供了多个方法来修改图片，例如：

直接将R、G、B、A传入，修改颜色所占比重:
```
public void setScale(float rScale, float gScale, float bScale,
                         float aScale) 
```
调节图片的对比度：
```
public void setSaturation(float sat)
```
颜色旋转（可以做色调效果）：
```
public void setRotate(int axis, float degrees)
```
修改对比度和亮度代码如下：
```

     // ... 省略若干代码

    private Paint paint;
    private ColorMatrixColorFilter colorMatrixColorFilter;
    private ColorMatrix colorMatrix;


    @Override
    protected void onDraw(Canvas canvas) {
        paint.reset();
        paint.setAntiAlias(true);
        if (colorMatrixColorFilter != null) {
               paint.setColorFilter(colorMatrixColorFilter);
         }
        canvas.drawBitmap(bitmap, null, rectF, paint);
        canvas.save();
    }
    /**
     * @param light 0f -2f
     */
    public void changeLight(float light) {
        colorMatrix = new ColorMatrix();
        colorMatrix.setScale(light, light, light, 1f);
        colorMatrixColorFilter = new ColorMatrixColorFilter(colorMatrix);
        invalidate();
    }

    /**
     * @param saturaction 0f - 2f
     */
    public void changeSaturation(float saturaction) {
        colorMatrix = new ColorMatrix();
        colorMatrix.setSaturation(saturaction);
        colorMatrixColorFilter = new ColorMatrixColorFilter(colorMatrix);
        invalidate();
    }

```
其中修改亮度的修改，就是将R、G、B三种颜色的比例不断增大。

![调节对比度和亮度](http://upload-images.jianshu.io/upload_images/1930161-58e235c05664d7d5.gif?imageMogr2/auto-orient/strip)

####3.3 BlurMaskFilter虚化边框

![MaskFilter](http://upload-images.jianshu.io/upload_images/1930161-41e84831ec4ec1ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

MaskFilter有两个子类：
BlurMaskFilter ：可实现模糊效果。
EmbossMaskFilter：可实现凸起效果。
这里用到了BlurMaskFilter 做虚化边框的效果：
BlurMaskFilter 的构造函数：
```
public BlurMaskFilter(float radius, Blur style)
```
radius：模糊的宽度。
style：枚举变量，提供不同的效果：
```
public enum Blur {
        /**
         * Blur inside and outside the original border.
         */
        NORMAL(0),

        /**
         * Draw solid inside the border, blur outside.
         */
        SOLID(1),

        /**
         * Draw nothing inside the border, blur outside.
         */
        OUTER(2),

        /**
         * Blur inside the border, draw nothing outside.
         */
        INNER(3);
        
        Blur(int value) {
            native_int = value;
        }
        final int native_int;
    }
```
使用模糊效果会用到几个关键类：BlurMaskFilter、Paint 、canvas。
代码步骤：
1、初始化BlurMaskFilter。
2、然后通过paint.setMaskFilter()法将BlurMaskFilter设置给paint。
3、通过canvas.drawBitmap()方法即可实现图片的模糊效果。（不仅仅是drawBitmap只要drawXX方法中传入了设置过BlurMaskFilter的paint都可实现模糊效果）
![虚化边框效果](http://upload-images.jianshu.io/upload_images/1930161-48812a087b076f4d.gif?imageMogr2/auto-orient/strip)
不过BlurMaskFilter遇到几个问题：
>1、 虚化的颜色由背景由图片的边缘颜色决定的，如果图片边缘的颜色为白色，那么虚化部分的颜色是白色的。
>2、 虚化宽度的最大值并没有找到。这个类调用的是native的方法，我不知道怎么去处理。
>3、保存图片之后有点失真。

####3.4 其他需要注意的问题
#####3.4.1、选择相册图片和使用相机的6.0权限处理
第一步：AndroidManifest.xml文件中申明权限：
```
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.CAMERA" />
```
动态申请权限：
```
    //申请相机权限
    protected boolean requestCameraPermiss() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.CAMERA}, PERMISSION_CAMERA);
            return false;
        }
        return true;
    }

    //申请读取文件权限
    protected boolean requestAlbumPermiss() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, PERMISSION_CAMERA);
            return false;
        }
        return true;

    }
```

#####3.4.2、Bitmap的压缩、生成新的bitmap等
######3.4.2.1 生成新的Bitmap
如果需要通过一个Bitmap生成一个新的Bitmap，需要将Bitmap画在一个新的Bitmap画布上。看代码注释。
```
    @Override
    public Bitmap getChangeBitmap() {
        //初始化一个Bitmap
        Bitmap bitmapAltered = Bitmap.createBitmap((int) rectF.right, (int) rectF.bottom, bitmap.getConfig());
        //将初始化的Bitmap当作画布
        Canvas canvas = new Canvas(bitmapAltered);
        Paint paint = new Paint();
        paint.setAntiAlias(true);
        paint.setColorFilter(colorMatrixColorFilter);
        if (drawType == DRAW_TYPE_MASK) {
            if (blurMaskFilter != null) {
                paint.setMaskFilter(blurMaskFilter);
            }
        }
      //将一个bitmap画在画布上（注：这里的bitmap是一个全局的变量，它就是一个普通的Bitmap）
        canvas.drawBitmap(bitmap, null, rectF, paint);
     //返回第一个Bitmap即可
        return bitmapAltered;
    }
```
######3.4.2.2 图片压缩的方法：
防止图片过大导致OOM，这里通过  BitmapFactory.Options 对图片进行压缩之后再展示。
```
  /**
     * 读取图片，按照缩放比保持长宽比例返回bitmap对象
     * <p>
     *
     * @param path  图片全路径
     * @param scale 缩放比例(1到10, 为2时，长和宽均缩放至原来的2分之1，为3时缩放至3分之1，以此类推)
     * @return Bitmap
     */
    public synchronized static Bitmap readBitmap(String path, int scale) {
        try {
            BitmapFactory.Options options = new BitmapFactory.Options();
            // 设置缩放比例
            options.inSampleSize = scale;
            // 设置为false,解析Bitmap对象加入到内存中
            options.inJustDecodeBounds = false;
            // 设置内存不足时，比bitmap对象可以被回收
            options.inPurgeable = true;
            options.inInputShareable = true;
            options.inPreferredConfig = Bitmap.Config.RGB_565;
            Bitmap bitmap = BitmapFactory.decodeFile(path, options);
            return bitmap;
        } catch (Exception e) {
            return null;
        }
    }
```
#####3.4.3、图片的分享
这里通过安卓自带的方法来做分享：
```
    Intent sendIntent = new Intent();
    sendIntent.setAction(Intent.ACTION_SEND);
    sendIntent.putExtra(Intent.EXTRA_STREAM, imageUrl);
    sendIntent.setType("image/");
    startActivity(Intent.createChooser(sendIntent, "分享图片"));
```
![分享图片](http://upload-images.jianshu.io/upload_images/1930161-31f3922cd29a7a52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3.5、推荐的链接
colorMatrx ：
http://www.jianshu.com/p/9a44d04f39fc
http://www.jianshu.com/p/a870fb9684d5
6.0权限：
http://www.jianshu.com/p/5e05691d9c76
http://blog.csdn.net/yanzhenjie1003/article/details/52503533/
分享图片：
http://www.jianshu.com/p/f790833e669d
http://www.jianshu.com/p/25c84ed9046d

源码地址：https://github.com/AxeChen/ColorFilter

**原创不易，如果大佬对这篇文章感兴趣，还望点个赞，给点鼓励！**
