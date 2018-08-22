---
layout:     post
title:      Android Material Design：使用Palette优化界面色彩搭配。
date:       2017-09-11
author:     陈再峰
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
	- Material Design
---


我写过一篇博客介绍了常用Material Design控件的使用。
http://www.jianshu.com/p/776cc6329fff
本想把大部分的Material Design的知识点写到一个博客中，结果无奈东西太多只好分开写。这篇博客介绍的是Palette调色板的使用。
源码地址：https://github.com/AxeChen/MaterialDesignSimple
![示例代码（由于提交中不慎修改了其他module的代码，所以用红框标出了）](http://upload-images.jianshu.io/upload_images/1930161-68734be72ccc9c68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1、Palette的基本理解
**Palette是调色版的意思，用它能获取到Bitmap中一些活跃的颜色，其他控件通过设置这些颜色来优化界面色彩搭配。**
**效果图：**
没有使用Palette调色时。Toolbar和TabLayout等控件是蓝色，和图片很明显不协调。
![没使用Palette调色效果图](http://upload-images.jianshu.io/upload_images/1930161-191b5a91d1693e13.gif?imageMogr2/auto-orient/strip)

使用Palette调色后。Toolbar、TabLayout状态栏等会根据图片展示比较搭配的颜色。
**注意：Palette并不只适用于我写的示例，Palette应用的情况很多，我这里只用作效果展示。**
![使用Palette调色效果图](http://upload-images.jianshu.io/upload_images/1930161-bed6eddbb3d12246.gif?imageMogr2/auto-orient/strip)


### 2、Palette的基本使用
Palette使用起来非常简单，没有比较复杂的步骤。
#### 2.1、导入依赖
```
    compile 'com.android.support:palette-v7:23.4.0'
```
#### 2.2、主要方法
调用Palette.from() 方法将bitmap传入，然后在回调中获取颜色值。
```
Palette.from(bitmap).generate(new Palette.PaletteAsyncListener() {
            @Override
            public void onGenerated(Palette palette) {
            }
        });
```
通过Palette 可以获取到一些颜色值。
```
 // 获取到柔和的深色的颜色（可传默认值）
 palette.getDarkMutedColor(Color.BLUE);
 // 获取到活跃的深色的颜色（可传默认值）
 palette.getDarkVibrantColor(Color.BLUE);
 // 获取到柔和的明亮的颜色（可传默认值）
 palette.getLightMutedColor(Color.BLUE);
 // 获取到活跃的明亮的颜色（可传默认值）
 palette.getLightVibrantColor(Color.BLUE);
 // 获取图片中最活跃的颜色（也可以说整个图片出现最多的颜色）（可传默认值）
 palette.getVibrantColor(Color.BLUE);
 // 获取图片中一个最柔和的颜色（可传默认值）
 palette.getMutedColor(Color.BLUE);
// ...  这里省略其他的方法。
```
以上我都是通过源码中的注释来获取这些方法的基本含义的。Palette的方法并不多，大部分都是返回颜色的方法，这里不再多介绍。
以上方法的效果图：
![效果图](http://upload-images.jianshu.io/upload_images/1930161-187e73b0d8845b90.gif?imageMogr2/auto-orient/strip)

#### 2.3、示例代码
示例代码中的TabLayout和Viewpager的联用，我就不介绍了。如果不知道TabLayout和Viewpager的联用可以看我的另一篇写Material Design控件的博客：http://www.jianshu.com/p/776cc6329fff
唯一注意的地方就是使用了不同的获取颜色的方法。
**palette.getVibrantSwatch()：**获取颜色样本。在这里做了非空判断，如果获取的颜色样本为空就从所有的样本中获取一个样本。
**vibrant.getRgb()：**从样本中获取颜色的RGB值。获取到RGB值之后可以直接给其他控件使用这个值，或者稍微调整这个值的颜色再使用。
```
Palette.from(bitmap).generate(new Palette.PaletteAsyncListener() {
            @Override
            public void onGenerated(Palette palette) {
                Palette.Swatch vibrant = palette.getVibrantSwatch();
                if (vibrant == null) {
                    for (Palette.Swatch swatch : palette.getSwatches()) {
                        vibrant = swatch;
                        break;
                    }
                }
                // 这样获取的颜色可以进行改变。
                int rbg = vibrant.getRgb();

                // ... 省略一些无关紧要的代码

                tabLayout.setBackgroundColor(rbg);
                toolbar.setBackgroundColor(rbg);
                if (Build.VERSION.SDK_INT > 21) {
                    Window window = getWindow();
                    //状态栏改变颜色。
                    int color = changeColor(rbg);
                    window.setStatusBarColor(color);
                }
            }
        });

  // 对获取到的RGB颜色进行修改。（涉及到位运算，我也不是很懂这块）
 private int changeColor(int rgb) {
      int red = rgb >> 16 & 0xFF;
      int green = rgb >> 8 & 0xFF;
      int blue = rgb & 0xFF;
      red = (int) Math.floor(red * (1 - 0.2));
      green = (int) Math.floor(green * (1 - 0.2));
      blue = (int) Math.floor(blue * (1 - 0.2));
      return Color.rgb(red, green, blue);
 }
```

### 3、最后
源码地址：[https://github.com/AxeChen/MaterialDesignSimple](https://github.com/AxeChen/MaterialDesignSimple)
