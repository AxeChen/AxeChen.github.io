会议上，产品经理突然眼里闪烁着一丝诡异的光，随后提出一个需求：Android的屏幕适配要从320X480的尺寸开始做适配，一直适配到1920X1080尺寸就行了。随后扬长而去。留下喷了一地老血的设计师和程序员。
设计师：我得切多少套图来着？
![我得切多少套图来着？](http://upload-images.jianshu.io/upload_images/1930161-43362a3841e50867.gif?imageMogr2/auto-orient/strip)
程序员：
![。。。](http://upload-images.jianshu.io/upload_images/1930161-f979727809239540.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


现在绝大多数APP就是通过使用多套图来做适配的：
这种适配的坏处：
1、设计师需要切多套图来应对需求，同时在切图中可能出现差错导致图片不清晰的情况。
2、程序员这边比较繁琐，同时随着图片的不断增多，apk会越来越大。
如何解决这种情况？
![我要开始装逼了](http://upload-images.jianshu.io/upload_images/1930161-d91ddef1ec6333c7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####SVG图片
**1，什么是 SVG ？ && SVG VS 位图**
Android 5.0发布的时候，Google提供了Vector的支持，即：Vector Drawable类。
SVG的介绍：http://blog.csdn.net/luoyanglizi/article/details/52589234


**2、SVG的好处**
Vector Drawable相对于普通的Drawable来说，有以下几个好处：
（1）Vector图像可以自动进行适配，不需要通过分辨率来设置不同的图片。
（2）Vector图像可以大幅减少图像的体积，同样一张图，用Vector来实现，可能只有PNG的几十分之一。
（3）使用简单，很多设计工具，都可以直接导出SVG图像，从而转换成Vector图像 功能强大。
（4）不用写很多代码就可以实现非常复杂的动画 成熟、稳定，前端已经非常广泛的进行使用了。

**3、Vector Drawable的基本定义和Path基本指令**
```
<vector xmlns:android="http://schemas.android.com/apk/res/android"
        android:width="24dp"
        android:height="24dp"
        android:viewportWidth="24.0"
        android:viewportHeight="24.0">
    <path
        android:fillColor="#FF000000"
        android:pathData="M6,18c0,0.55 0.45,1 1,1h1v3.5c0,0.83 0.67,1.5 1.5,1.5s1.5,-0.67 1.5,-1.5L11,19h2v3.5c0,0.83 0.67,1.5 1.5,1.5s1.5,-0.67 1.5,-1.5L16,19h1c0.55,0 1,-0.45 1,-1L18,8L6,8v10zM3.5,8C2.67,8 2,8.67 2,9.5v7c0,0.83 0.67,1.5 1.5,1.5S5,17.33 5,16.5v-7C5,8.67 4.33,8 3.5,8zM20.5,8c-0.83,0 -1.5,0.67 -1.5,1.5v7c0,0.83 0.67,1.5 1.5,1.5s1.5,-0.67 1.5,-1.5v-7c0,-0.83 -0.67,-1.5 -1.5,-1.5zM15.53,2.16l1.3,-1.3c0.2,-0.2 0.2,-0.51 0,-0.71 -0.2,-0.2 -0.51,-0.2 -0.71,0l-1.48,1.48C13.85,1.23 12.95,1 12,1c-0.96,0 -1.86,0.23 -2.66,0.63L7.85,0.15c-0.2,-0.2 -0.51,-0.2 -0.71,0 -0.2,0.2 -0.2,0.51 0,0.71l1.31,1.31C6.97,3.26 6,5.01 6,7h12c0,-1.99 -0.97,-3.75 -2.47,-4.84zM10,5L9,5L9,4h1v1zM15,5h-1L14,4h1v1z"/>
</vector>

```

Path指令解析如下所示：
```
M = moveto(M X,Y) ：将画笔移动到指定的坐标位置，相当于 android Path 里的moveTo()
L = lineto(L X,Y) ：画直线到指定的坐标位置，相当于 android Path 里的lineTo()
H = horizontal lineto(H X)：画水平线到指定的X坐标位置 
V = vertical lineto(V Y)：画垂直线到指定的Y坐标位置 
C = curveto(C X1,Y1,X2,Y2,ENDX,ENDY)：三阶贝赛曲线 
S = smooth curveto(S X2,Y2,ENDX,ENDY) 同样三阶贝塞尔曲线，更平滑 
Q = quadratic Belzier curve(Q X,Y,ENDX,ENDY)：二阶贝赛曲线 
T = smooth quadratic Belzier curveto(T ENDX,ENDY)：映射 同样二阶贝塞尔曲线，更平滑 
A = elliptical Arc(A RX,RY,XROTATION,FLAG1,FLAG2,X,Y)：弧线 ，相当于arcTo()
Z = closepath()：关闭路径（会自动绘制链接起点和终点）
’M’处理时，只是移动了画笔， 没有画任何东西。
```

注意：1.关于这些语法，开发者不需要全部精通，而是能够看懂即可，这些path标签及数据生成都可以交给工具来实现。（一般美工来帮你搞定！PS、Illustrator等等都支持导出SVG图片）

**5、写一个例子**
在这里通过Android studio自带的工具生成一个SVG图片：
首先右键res-->new-->vector asset
![第一步：找到Vector Asset](http://upload-images.jianshu.io/upload_images/1930161-63254cf28703ad63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击vector asset后会出现Vector Drawable的一些属性配置的信息，这里直接next吧。
![第二部：一些配置](http://upload-images.jianshu.io/upload_images/1930161-1401600271ce0d45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击next之后就可以在drawable里面看到生成的Vector Drawable的xml文件了
![最后：生成SVG图片](http://upload-images.jianshu.io/upload_images/1930161-3a48a81ff8debf57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


添加四个ImageView，ImageView的宽高从50dp到200dp。
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <ImageView
        android:layout_width="50dp"
        android:layout_height="50dp"
        app:srcCompat="@drawable/ic_android_black_24dp" />

    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        app:srcCompat="@drawable/ic_android_black_24dp" />

    <ImageView
        android:layout_width="150dp"
        android:layout_height="150dp"
        app:srcCompat="@drawable/ic_android_black_24dp" />

    <ImageView
        android:layout_width="200dp"
        android:layout_height="200dp"
        app:srcCompat="@drawable/ic_android_black_24dp" />


</LinearLayout>
```

这里设置Vector Drawable的步骤：
首先要添加   ``` xmlns:app="http://schemas.android.com/apk/res-auto"```命名空间
然后通过 ```app:srcCompat```来设置。
在使用SVG图片是最好先在当前app的build.gradle中添加如下配置：
```
android {
    defaultConfig {
       //需要添加的配置
        vectorDrawables.useSupportLibrary = true
    }
}
```
如果**没有**添加以上配置，每次使用app:srcCompat时需要添加```app:ignore="VectorDrawableCompat"```才能编译通过：
```
    <ImageView
        android:layout_width="50dp"
        android:layout_height="50dp"
        app:srcCompat="@drawable/ic_android_black_24dp"
        app:ignore="VectorDrawableCompat" />
```
运行程序结果：

![握草，还有这种操作？](http://upload-images.jianshu.io/upload_images/1930161-21c6b81de539b4e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出图片没有出现模糊。由于dp是会根据不同的屏幕大小和密度自动改变的，所以就达到适配的效果了。
当然也可以在dimens中根据不同的屏幕设置不同的大小进行适配。
上面已经证明了Vector Drawable拉伸是不会失真的，怎样去适配都可以。

**6、兼容问题**
Vector Drawable是Android5.0发布时Google提供的支持，所以5.0以上是直接使用的。
那么5.0以下的怎么做兼容呢？
1、使用Android Studio 2.2以上的版本。
2、添加defaultConfig (上面也有提到)：
```
defaultConfig {
		vectorDrawables.useSupportLibrary = true
	}
```
3、添加 ompile 'com.android.support:appcompat-v7:25.3.1'   **需要是23.2 版本以上的**
3、Activity需要继承与AppCompatActivity
4、使用在Actvity前面添加一个配置:
```
static {
			AppCompatDelegate.setCompatVectorFromResourcesEnabled(true);
	}
```
5、注意使用方法：比如上面例子中的ImageView的使用方法


**虽然兼容起来有点麻烦，但是以后可能会成为主流，所以是时候学习Vector Drawable这个东西了**
**7、一些案例**
微信已经用到了这个技术：
http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=207863967&idx=1&sn=3d7b07d528f38e9f812e8df7df1e3322&scene=4#wechat_redirect

**8、一些网址**
参考的博客：http://blog.csdn.net/luoyanglizi/article/details/52589234
一些生成SVG的网址：
http://inloop.github.io/svg2android/ 
http://editor.method.ac/
通过SVG图片生成VectorDrawable.xml网址：
Android studio也可以生成
http://inloop.github.io/svg2android/ 
SVG图片下载的地址：
http://www.iconfont.cn/plus 
http://www.flaticon.com/

**原创不易，如果对这篇文章感兴趣，求大佬点个赞**
