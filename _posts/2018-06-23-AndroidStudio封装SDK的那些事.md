---
layout:     post
title:      AndroidStudio封装SDK的那些事
subtitle:   想知道AndroidStudio如何封装SDK吗？想知道AndroidStudio和Eclipse如何接入SDK吗？
date:       2018-06-23
author:     陈再峰
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
    - SDK
---

首先SDK是提供给别人调用的工具。所以常见的SDK都是以jar包，so库，aar包等方式导入APP项目中。然后提供一些公开的API供接入方调用。所以在Androidstudio中如果需要生成jar或者aar，就需要将module变成library。
#### 1、AndroidStudio生成library
在这里介绍AndroidStudio两种生成library的方式。
##### 1.1、两种生成library的方式
###### 新建library module。
![image.png](https://upload-images.jianshu.io/upload_images/1930161-bba4e6334023588a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这种会直接生成可编译成jar和aar的module。

###### 新建Android项目，然后修改app下的build.gradle
![image.png](https://upload-images.jianshu.io/upload_images/1930161-890e340522780980.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将  ```apply plugin: 'com.android.application'```修改成```apply plugin: 'com.android.library'```
然后去掉``` applicationId "com.mg.axe.helloworld"```就把可运行的Android module变成了一个library module。
**注意：这种方式在编译前一定要做以下事情**
* 删除自定义的Application和在AndroidManifest.xml的配置。
* 去点入口的Activity，否则在Android Studio接入时会生成两个图标入口。
    

##### 1.1、使用gradle所带的命令编译
这些命令可以自己在控制台使用，可以直接点开右上角的Gradle直接使用。
![image.png](https://upload-images.jianshu.io/upload_images/1930161-c1bb96df76855c35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*  **assembleRelease**&**assembleDebug**     

![image.png](https://upload-images.jianshu.io/upload_images/1930161-0ba017df99ed59d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在build下的assembleRelease和assembleDebug都可以生成aar包。这边和APP开发很相似，可以在buildTypes下对release包做混淆等等操作。

如果编译的命令执行完毕，可以在当前module下的build文件下找到编译好的.aar文件。
![image.png](https://upload-images.jianshu.io/upload_images/1930161-ae5355ade2502e4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果需要jar包，则只需将这个aar文件解压即可。
![image.png](https://upload-images.jianshu.io/upload_images/1930161-91e77aff157042f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
classes.jar就是编译成jar的class文件。
##### 1.2、aar和jar

* .aar是适用于AndroidStudio的接入方式，不需要过多的考虑当SDK存在界面，图片等资源文件的情况。解压aar也可以看到，aar是一个将源码（jar）和资源文件都打包好的文件。当然也可以在eclipse中使用，前提是eclipse需要安装gradle编译环境。

* jar只包含编译好的源代码，如果SDK包含资源文件，则需要额外导入，适用于eclipse导入；AndroidStudio也同样适用，不过当SDK包含资源文件时，导入aar将会更方便。

#### 2、两种接入方式
一般情况接入方式为AndroidStudio和Eclipse。其他的接入方式就不考虑了，可能大同小异，最主要的是其他的接入方式我也不会。
![手动滑稽](https://upload-images.jianshu.io/upload_images/1930161-aee57be922bc3c13.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 2.1、AndroidStudio接入方式
这里只介绍.aar的接入方式，AndroidStudio接入jar方式就不做介绍。
* 将.aar文件复制到项目的libs中。
* 并在app下的build.gradle中的android中添加如下代码   
```
repositories{
        flatDir {
            dirs 'libs'
        }
    }
```
* 在dependencies中添加依赖的代码
```
  // implementation(name: 'aar包的名字', ext: 'aar')
  implementation(name: 'game_sdk', ext: 'aar')
```
然后点击同步（Sync Now）,就成功的将.arr导入项目了。

可以在External Libraries中找到导入的aar依赖。
![image.png](https://upload-images.jianshu.io/upload_images/1930161-9014cf4f859fa5a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1930161-2224b0905f66ad49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点开aar，可以看(源代码)jar和（资源文件）res。

##### 2.2、Eclipse接入方式
eclipse一般是接入jar包的方式接入SDK，当SDK存在界面、资源文件时，接入方式比AndroidStudio接入aar稍微麻烦点，需要将jar包和资源文件分开导入。
* 解压aar文件。
* 将jar包复制到libs文件加下，并添加依赖（add to path）  。
* 如果有资源文件，则需要将res下的资源文件复制到项目对应的位置。
* 如果SDK用到了Activity等组件，还需去注册等，这些都应在SDK接入文档中指明清楚。

##### 2.3、两种接入方式都需要注意的问题
在SDK中声明的权限，制定的Android版本范围等都要在SDK接入文档中指明清楚。

#### 3、可能踩的坑
##### 3.1、资源文件无法获取的问题。
如果编译好的jar中使用了资源文件，然后使用了R.xx.xx这样的代码，可能会出现这样的异常。
```
java.lang.NoClassDefFoundError: Failed resolution of: Lcom/ysyc/axechen/R$id
```
找不到id。最后是参照开源的TypeSDK才解决了这个问题。通过如下的方法去寻找id。
```
public class GetResId {
    public static int getId(Context context, String paramString1, String paramString2) {
        return context.getResources().getIdentifier(paramString2, paramString1, context.getPackageName());
    }
}
```
加载布局和控件的方法：
```
// 获取布局id
GetResId.getId(this, "layout", "activity_main")

// 获取控件id
GetResId.getId(this, "id", "login")
```

##### 3.2、三方包冲突问题
如果SDK用到了三方库，然后接入方的项目中也用到了同样的三方库，那么当编译的时候就会出现类冲突，无法编译通过。这个时候就要求在编译SDK时不要将三方的依赖编译到SDK的jar中。那么在添加依赖时需要使用compileOnly关键字。
```
compileOnly files('libs/gson-2.8.5.jar')
```
或者
```
compileOnly 'com.google.code.gson:gson:2.8.5'
```
这样才不会将引入的依赖编译到SDK的jar中，这个时候需要接入方导入这些依赖，当然SDK的接入文档要详细说明。

##### 3.3、请使用最平常的api和习惯
最好不去使用一些新的特性。如果接入方没有使用到这些特性，可能编译无法通过，尤其是eclipse接入时会出现更多问题。我遇到的问题：我在编译SDK时就是因为使用了lamada表达式导致eclipse无法编译通过。

#### 4、混淆
SDK的混淆和做app的混淆是一样的。
```
 buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```
在混淆的时候，如果使用了三方库，三方库混淆的要求同样需要加上混淆。
**如果接入方需要做混淆，请记住加上SDK的混淆要求和三方库的混淆要求。免得SDK的代码混淆之后又被接入方混淆导致出错。**

#### 5、关于SDK的其他解决方案
实际上，用原生的界面做SDK并不是非常好的解决方案，主要是不利于SDK的更新和跨平台。最好的方式是加载H5，更新起来更方便，SDK实现起来更简单。

#### 6、一些开源的SDK
https://github.com/typesdk/TypeSDK
https://github.com/zuowutan/ShareGameSdk

**如果这篇文章对你有帮助，还请点个赞再走吧:)**
