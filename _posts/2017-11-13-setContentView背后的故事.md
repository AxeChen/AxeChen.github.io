---
layout:     post
title:      setContentView背后的故事
subtitle:   一步一步解析setContentView方法，探究不同情况下的Activity的层数。
date:       2017-11-13
author:     陈再峰
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - Activity
    - 源码解析
---

Android程序员都知道Activity调用setContentView的方法是将xml布局文件加载到Activity中。那么：
**调用setContentVIew后到底是怎样将xml布局文件加载到Activity中去的？**
**继承AppCompatActivity和继承Activity的Activity布局结构有什么不同？**
接下来从源码的角度分析setContentView背后的逻辑。

因为分析源码，所以下面的内容可能会引起深度不适。

![吐血](http://upload-images.jianshu.io/upload_images/1930161-e2492752cc0538a4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**阅读源码的建议：**阅读源码一定要带着目的去阅读。比如说：**调用setContentVIew后到底是怎样将xml布局文件加载到Activity中去的？**然后针对这个问题去寻找答案。如果没有目的性的看源码会非常艰难。还有就是看源码真的很累。在写这篇文章时，源码真的是看了一遍又一遍。所以贵在坚持吧。

#### 1、从Activity的setContentVIew开始
API22之后，出现了AppCompatActivity。所有的Activity默认继承它，在AppCompatActivity中使用代理模式做了许多兼容，和一些新的特性的添加。但AppCompatActivity的基类依旧是Activity。所谓“万变不离其宗”，先看下Activity的setContentVIew，然后再去看AppCompatActivity到底做了怎样的兼容。  

#### 1.1、setContentView的作用
新建一个```MainActivity``` 继承```Activity```：
```
  public class MainActivity extends Activity {
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
      }
  }
```
跳转到Activiy中去看setContentView：
```
    /**
     * Set the activity content from a layout resource.  The resource will be
     * inflated, adding all top-level views to the activity.
     *
     * @param layoutResID Resource ID to be inflated.
     *
     * @see #setContentView(android.view.View)
     * @see #setContentView(android.view.View, android.view.ViewGroup.LayoutParams)
     */
    public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }
```
注意看这段注释：
```
    * Set the activity content from a layout resource.  The resource will be
    * inflated, adding all top-level views to the activity.
```
这段注释的含义：
>调用这个方法会将资源文件中的xml布局文件，把所有的顶级View加载到Activity中去。

这个和我们平时的理解一样：将资源文件中的布局文件加载到Activity中去。
**请注意，这里有个顶级View的概念，往下看我们会讲到。**

> 调用setContentView的作用就是将资源文件中的布局文件加载到Activity中去。（直接根据注释下结论确实有点生硬，不过往下看代码，具体去了解调用setContentView是如何将资源文件加载到Activity中去的。）

****
#### 1.2、最顶层Window——PhoneWindow

继续看代码。```Activity```的```setContentView()```中又调用了```getWindow()```的```setContentView()```方法。
```
 getWindow().setContentView(view);
```
在```Activity```中，跳转到```getWindow()```看这个方法：
```
    /**
     * Retrieve the current {@link android.view.Window} for the activity.
     * This can be used to directly access parts of the Window API that
     * are not available through Activity/Screen.
     *
     * @return Window The current window, or null if the activity is not
     *         visual.
     */
    public Window getWindow() {
        return mWindow;
    }
```
可以看到```getWindow()```这个方法返回了一个```Window```。
跳转到```Window```类，看下```Window```到底是什么：
```
  /**
   * Abstract base class for a top-level window look and behavior policy.  An
   * instance of this class should be used as the top-level view added to the
   * window manager. It provides standard UI policies such as a background, title
   * area, default key processing, etc.
   *
   * <p>The only existing implementation of this abstract class is
   * android.view.PhoneWindow, which you should instantiate when needing a
   * Window.
   */
  public abstract class Window { ... }
```
仔细看如下注释：
```
 * Abstract base class for a top-level window look and behavior policy.  An
 * instance of this class should be used as the top-level view added to the
 * window manager. It provides standard UI policies such as a background, title
 * area, default key processing, etc.
```
这段注释的含义：

>用于顶级窗口外观和行为策略的抽象基类。这个类的实例应该被作为顶层视图添加到窗口管理器。它提供了标准的UI策略，例如背景、标题 区域、默认密钥处理等。 他只有一个实现类：android.view.PhoneWindow。

> ```Window```是最顶层Window的抽象基类。它提供了```Window```外观，行为等的策略。它只有一个实现类```PhoneWindow```。

**注意：上面提到了最顶层View，这里提到了最顶层Window，两者区别是非常大的，后面会讲到两者的关系。**

既然Window只有一个实现类```PhoneWindow```，那么```Activity```中的：
```
  getWindow().setContentView(layoutResID);
```
实际上调用的就是```PhoneWindow```的```setContentView```方法！！
****
#### 1.3、PhoneWindow的setContentView
跳转到```PhoneWindow```的```setContentView```方法：
```
  @Override
  public void setContentView(int layoutResID) {
      // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
      // decor, when theme attributes and the like are crystalized. Do not check the feature
      // before this happens.
      if (mContentParent == null) {
          installDecor();
      } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
          mContentParent.removeAllViews();
      }
      if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
          final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                  getContext());
          transitionTo(newScene);
      } else {
          mLayoutInflater.inflate(layoutResID, mContentParent);
      }
      mContentParent.requestApplyInsets();
      final Callback cb = getCallback();
      if (cb != null && !isDestroyed()) {
          cb.onContentChanged();
      }
      mContentParentExplicitlySet = true;
  }
```
首先大概看下这个方法中的代码，前面是```mContentParent```初始化的逻辑。
后面的代码是将Id为```layoutResID```资源文件的布局加载到```mContentParent```中去。因为里面有一句非常重要的代码：``` mLayoutInflater.inflate(layoutResID, mContentParent);```
```
    if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                getContext());
        transitionTo(newScene);
    } else {
        mLayoutInflater.inflate(layoutResID, mContentParent);
    }
```
只要是Android程序员一定对```mLayoutInflater.inflate```不陌生。   
这里还有一个是否有过渡动画的判断```if (hasFeature(FEATURE_CONTENT_TRANSITIONS))```，
如果是```true```，最后会调用```transitionTo(newScene)```。这个方法最后也是将Id为```layoutResID```资源文件的布局加载到```mContentParent```中去。从```transitionTo```开始一直跳转进去，最后在Scene类的enter方法中找到答案：
```
public void enter() {
        // Apply layout change, if any
        if (mLayoutId > 0 || mLayout != null) {
            // empty out parent container before adding to it
            getSceneRoot().removeAllViews();
            if (mLayoutId > 0) {
                // 重要的代码
                LayoutInflater.from(mContext).inflate(mLayoutId, mSceneRoot);
            } else {
                mSceneRoot.addView(mLayout);
            }
        }
      // ... 这里省略部分代码。
    }
```
了解mLayoutId, mSceneRoot的初始化就能知道答案了，这里就不多做解释。

> PhoneWindow的setContentView方法会将xml资源文件加载到mContentParent中。

那么问题来了，```mContentParent```是什么？它又是如何初始化的？

****
#### 1.4、PhoneWindow的mContentParent和DecorView

首先看```mContentParent```是什么。在```PhoneWindow```中能找到```mContentParent```的解释：
```
    // This is the view in which the window contents are placed. It is either
    // mDecor itself, or a child of mDecor where the contents go.
    ViewGroup mContentParent;
```
这段注释的意思是：
> 这是一个放置Window内容的View。他不是decorView自己，就是decorView的子View。（**这里先不管mDecor是什么，后面会提到。**）

继续看代码。
在```PhoneWindow```中的```setContentView```中看下这个判断：
```
   if (mContentParent == null) {
        installDecor();
   } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
       mContentParent.removeAllViews();
   }
```

既然这里是```mContentParent```的非空判断，那么```installDecor()```方法一定是初始化```mContentParent```的方法。

跳转到```installDecor()```中看是如何初始化```mContentParent```的：

****

#### 1.4.1、PhoneWindow初始化DecorView
```installDecor()```在出事化```mContentParent```前先初始化了```DecorView```。所以先看下```DecorView```的初始化以及```DecorView```和```Window```的关系。
```
 private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        } else {
            mDecor.setWindow(this);
        }
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor);

        ///...  省略若干代码
        }
    }
```
首先是```mDecor```的初始化。```generateDecor```是初始化```mDecor```的方法。
```
   if (mDecor == null) {
           mDecor = generateDecor(-1);
   }
```
在```PhoneWindow```的中，可以看到```mDecor```的定义：
```
    // This is the top-level view of the window, containing the window decor.
    private DecorView mDecor;
```
这段注释的含义是：

> 这是Window的顶层View，包含了Window的装饰。

跳转到```DecorView```看```DecorView```的具体代码：
```
public class DecorView extends FrameLayout implements RootViewSurfaceTaker, WindowCallbacks {
  //... 省略很多很多代码
}
```
根据以上可以知道```DecorView```的作用：

> DecorView是一个FrameLayout，是最顶层的View。包含Window（顶级窗口）的装饰（比如大小，是否透明等等属性）会体现在这个View上。这里可以大概知道顶级窗口Window（PhoneWindow）和顶级View（DecorView）的关系：顶级窗口Window（PhoneWindow）包含的状态属性，会在顶级View（DecorView）体现出来，比如窗口大小，背景等属性。（当然下面的代码也会体现出这点）

回到```PhoneWindow```，在```PhoneWindow```中看下```DecorView```的初始化方法```generateDecor()```:
```
   protected DecorView generateDecor(int featureId) {
      // System process doesn't have application context and in that case we need to directly use
      // the context we have. Otherwise we want the application context, so we don't cling to the
      // activity.
      Context context;
      if (mUseDecorContext) {
          Context applicationContext = getContext().getApplicationContext();
          if (applicationContext == null) {
              context = getContext();
          } else {
              context = new DecorContext(applicationContext, getContext().getResources());
              // 设置主题
              if (mTheme != -1) {
                  context.setTheme(mTheme);
              }
          }
      } else {
          context = getContext();
      }
      // new 一个DecorVIew
      return new DecorView(context, featureId, this, getAttributes());
  }
```
最后有这样的代码：
```
  new DecorView(context, featureId, this, getAttributes());
```
那么```generateDecor()```就是初始化```DecorView ```。
里面的第三个参数，就是```PhoneWindow```。初始化```DecorView```时会将```PhoneWindow```传进去。
看下这个完整的if判断：
```
  if (mDecor == null) {
      mDecor = generateDecor(-1);
  } else {
      mDecor.setWindow(this);
  }

```
如果```DecorView```不为空，会调用```setWindow()```。传入的this也是```PhoneWindow```。同时跳转到```DecorVIew```中可以看到类似如下代码：
```
public void setWindowBackground(Drawable drawable) {
        if (getBackground() != drawable) {
            setBackgroundDrawable(drawable);
            if (drawable != null) {
                mResizingBackgroundDrawable = enforceNonTranslucentBackground(drawable,
                        mWindow.isTranslucent() || mWindow.isShowingWallpaper());
            } else {
                mResizingBackgroundDrawable = getResizingBackgroundDrawable(
                        getContext(), 0, mWindow.mBackgroundFallbackResource,
                        mWindow.isTranslucent() || mWindow.isShowingWallpaper());
            }
            if (mResizingBackgroundDrawable != null) {
                mResizingBackgroundDrawable.getPadding(mBackgroundPadding);
            } else {
                mBackgroundPadding.setEmpty();
            }
            drawableChanged();
        }
    }
```
或者是这样的代码：
```
    final WindowManager.LayoutParams attrs = mWindow.getAttributes();
    if ((attrs.flags & FLAG_LAYOUT_IN_SCREEN) == 0) {
        if (attrs.height == WindowManager.LayoutParams.WRAP_CONTENT) {
        }
    }
```

> 以上代码是从```Window```里面获取属性来初始化```DecorView```的属性，或者根据```Window```的属性来设置```DecorView```的属性。这样的代码还有很多，这里不一一贴出，那么上面提到的 ```window```的装饰（比如大小，是否透明等等属性）会体现在```DecorView```上就解释得通了。（**这里也说明了Window和DecorVIew的关系**）

****

#### 1.4.2、PhoneWindow初始化mContentParent 

回到```PhoneWindow```类。初始化```DecorView```之后，继续看代码了解mContentParent的初始化：

```
 if (mContentParent == null) {
          mContentParent = generateLayout(mDecor);
 }
```
可以看到，这里```generateLayout()```方法初始化了```mContentParent```。
```generateLayout()```这个方法太长了。截取的几段关键的代码：

```
 protected ViewGroup generateLayout(DecorView decor) {
        // Apply data from current theme.

        // 拿window设置的Style
        TypedArray a = getWindowStyle();

        // 是不是浮动的Window ，例如Dialog等
        mIsFloating = a.getBoolean(R.styleable.Window_windowIsFloating, false);
        int flagsToUpdate = (FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR)
                & (~getForcedWindowFlags());
        if (mIsFloating) {
            setLayout(WRAP_CONTENT, WRAP_CONTENT);
            setFlags(0, flagsToUpdate);
        } else {
            setFlags(FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR, flagsToUpdate);
        }
// ... ... 省略若干代码。
}
```
首先第一句代码就是：
```
    TypedArray a = getWindowStyle();
```
```TypedArray```，如果写过自定义控件，对于自定义属性的获取绝对不会陌生。
看下```getWindowStyle()```的具体实现：
```
    /**
     * Return the {@link android.R.styleable#Window} attributes from this
     * window's theme.
     */
    public final TypedArray getWindowStyle() {
        synchronized (this) {
            if (mWindowStyle == null) {
                mWindowStyle = mContext.obtainStyledAttributes(
                        com.android.internal.R.styleable.Window);
            }
            return mWindowStyle;
        }
    }
```
```getWindowStyle()```这个方法就是获取window的属性然后返回。
如果写过自定义控件，在写自定义控件时，获取到自定义属性之后，一般是给自定义View的成员变量赋初始值。来确定自定义View属性的值。这是编写自定义View的一个思路。

**当然！**```PhoneWindow```也是这么干的。继续看下面的代码。
这里贴出一些代表性的，稍微熟悉点的代码，看看获取到window的style之后都干了些什么。
```
  // 获取window的属性
   TypedArray a = getWindowStyle();

  // 是不是浮动的Window ，例如Dialog等
  mIsFloating = a.getBoolean(R.styleable.Window_windowIsFloating, false);
        
  // 是不是设置了noTitle的style
  if (a.getBoolean(R.styleable.Window_windowNoTitle, false)) {}
   
  // 是不是透明的Window
  mIsTranslucent = a.getBoolean(R.styleable.Window_windowIsTranslucent, false);
```
以上的代码从```TypedArray```获取一些值来判断：
* window是不是可以浮动的window。比如dialog等；
* 判断是不是设置了Title；
* 判断是不是透明的window；
* ... ...

然后将这些值复制给```PhoneWindow```的成员变量。
这个完全和自定View时获取自定义属性和赋初始值的逻辑一模一样！

获取完Window的属性之后干了什么呢？

继续往下看代码：
```
    int layoutResource;
    // 拿到设置属性，然后去加载不同的XML。
    int features = getLocalFeatures();
    // System.out.println("Features: 0x" + Integer.toHexString(features));
    if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
        layoutResource = R.layout.screen_swipe_dismiss;
        setCloseOnSwipeEnabled(true);
    } else if (){
        // 省略若干代码... ...
    } else {
        // Embedded, so no decoration is needed.
        layoutResource = R.layout.screen_simple;
        // System.out.println("Simple!");
    }
   // 将加载的布局文件加载到DecorView中去 
       mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
```
上面的代码首先定义一个```layoutResource```，然后根据```features```的值加载不同的布局文件（features也是根据Window设置的属性获取到的值）。然后将布局文件id赋值给```layoutResource```。接下来的代码回将```layoutResource```加载到```DecorView```中去。

**请注意：这里虽然讲的是初始化mContentView，但是这个布局确实是加载到DecorView中去。**

看下这段代码：
```
  // 将加载的布局文件加载到DecorView中去 
  mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
```
跳转到DecorView看onResourcesLoaded的具体实现：
```
  void onResourcesLoaded(LayoutInflater inflater, int layoutResource) {
       mStackId = getStackId();
      if (mBackdropFrameRenderer != null) {
          loadBackgroundDrawablesIfNeeded();
          mBackdropFrameRenderer.onResourcesLoaded(
                  this, mResizingBackgroundDrawable, mCaptionBackgroundDrawable,
                  mUserCaptionBackgroundDrawable, getCurrentColor(mStatusColorViewState),
                  getCurrentColor(mNavigationColorViewState));
      }
      mDecorCaptionView = createDecorCaptionView(inflater);
      final View root = inflater.inflate(layoutResource, null);
      if (mDecorCaptionView != null) {
          if (mDecorCaptionView.getParent() == null) {
              addView(mDecorCaptionView,
                      new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
          }
          mDecorCaptionView.addView(root,
                  new ViewGroup.MarginLayoutParams(MATCH_PARENT, MATCH_PARENT));
      } else {
          // Put it below the color views.
          addView(root, 0, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
      }
      mContentRoot = (ViewGroup) root;
      initializeElevation();
  }
```
又是```mLayoutInflater```!又看到了熟悉的面孔，一看到```LayoutInflater```应该就能想到肯定是和加载布局有关。看下```onResourcesLoaded()```的思路：
首先将根据```layoutResource```加载成一个View：
```
final View root = inflater.inflate(layoutResource, null);
```
然后调用```addView()```将这个```View```添加到```DecorView```中去！
```
addView(root, 0, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
```
这段代码面还有```DecorCaptionView```的判断。
```DecorCaptionView```是什么呢？找到初始化```DecorCaptionView```的方法.
```
// Free floating overlapping windows require a caption.
    private DecorCaptionView createDecorCaptionView(LayoutInflater inflater){
  // ... 省略若干代码
}
```
这段注释的意思是：
> 自由浮动的重叠窗口需要一个标题。

自由浮动的窗口？这种情况我在开发中用的比较少。可能是Dialog？所以我暂时不考虑这种情况。只考虑普通的Activity的加载。
那么加载了什么样的布局到了```DecorView```中去呢 ？
回到```PhoneWindow``类的```generateLayout()```看下最简单的这个布局 ```R.layout.screen_simple```
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    android:orientation="vertical">
    <ViewStub android:id="@+id/action_mode_bar_stub"
              android:inflatedId="@+id/action_mode_bar"
              android:layout="@layout/action_mode_bar"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:theme="?attr/actionBarTheme" />
    <FrameLayout
         android:id="@android:id/content"
         android:layout_width="match_parent"
         android:layout_height="match_parent"
         android:foregroundInsidePadding="false"
         android:foregroundGravity="fill_horizontal|top"
         android:foreground="?android:attr/windowContentOverlay" />
</LinearLayout>
```
这个布局是```LinearLayout```，```LinearLayout```中有个```ViewStub```和```FrameLayout```。
```android:id="@android:id/content"```这个id好像似曾相识。相信有人用过```findViewById(android.R.id.content)```来获取Activity内容的根布局。 

 （如果你手中有本《安卓艺术开发探索》请打开第176页，这里面也有对android.R.id.content的简单的说明）。ps：这个是我突然想起来的。

如果没用到过这个id。请记住它，下面马上就用到了。

继续看generateLayout的代码：
```
// 找到刚刚的Content
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
        if (contentParent == null) {
            throw new RuntimeException("Window couldn't find content container view");
        }
        // ... ... 省略若干代码
        // 最后返回这个contentParent
        return contentParent;
```
看下这个Id：ID_ANDROID_CONTENT
```
    /**
     * The ID that the main layout in the XML layout file should have.
     */
    public static final int ID_ANDROID_CONTENT = com.android.internal.R.id.content;
```
没错！这个Id就是前面讲到的FrameLayout的id。然后通过findById的方法获取到一个ViewGroup。这个ViewGroup就是```mContentParent```。最后返回这个ViewGroup。

****

#### 1.4.3 installDecor()方法的总结
```installDecor()```里面包含了大量的逻辑。
开始初始化了```DecorView```。  
然后初始化```mContentParent```。
在初始化```mContentParent```时，又看到：
获取window的属性并赋值给```PhoneWindow```的逻辑。  
看到了根据window的属性加载了不同的布局，并加载到给```DecorView```的逻辑。  


最后看到了通过findViewById的方法获取到了```mContentParent```并且返回。

****

#### 1.4.4 回到PhoneWindow的setContentView
这个逻辑在前面也写过了。再回顾下PhoneWindow的setContentView的逻辑。
```
 @Override
    public void setContentView(int layoutResID) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized. Do not check the feature
        // before this happens.
        if (mContentParent == null) {
            installDecor();
            //不为空诗先RemoviewAllViews
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        // 是否存在动画
        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            // 吧资源文件添加到 mContentParent 中去
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
```
关键代码：
```
  // 吧资源文件添加到 mContentParent 中去
  mLayoutInflater.inflate(layoutResID, mContentParent);
```
前面讲了初始化```mContentParent```的逻辑，这里将布局文件加载到了```mContentParent```去。
****
#### 1.4.5 流程回顾

**第一步：初始化DecorView**
```
 generateDecor(-1);
```
**第二步：初始化mContentParent**
```
generateLayout(mDecor);
```
在初始化```mContentParent```的同时，看到了初始化```PhoneWindow```的属性，加载了一个```LinearLayout```到```Decoview```中。
**第三步：将布局加载到mContentParent中**
```
 public void setContentView(int layoutResID) {
       mLayoutInflater.inflate(layoutResID, mContentParent);
 }
```

画图来表示一下继承Activity时的布局结构：

![继承Activity时的布局结构](http://upload-images.jianshu.io/upload_images/1930161-a81438b7c42a100b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 回顾下前面的东西
如果以上的内容已经看吐了，确实我写这个玩意也写吐了。但是还得继续写下去，因为还有继承AppCompatActivity的setContentView。
如果你能跟着上面博客看完，并且能看懂的，我表示非常感谢。先不要急着看下面的内容。先给自己一杯热茶或者咖啡，走动看看窗外。回想下```setContentView()```之后的流程，回想一下DecorView，mContentPerant是如何初始化的。能弄懂以上才能更容易看懂继承AppCompatActivity的setContentView的流程。
****
#### 2、AppCompatActivity的setContentVIew
接下来看下继承AppCompatActivity的setContentView。
```
  public class MainActivity extends AppCompatActivity {
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
      }
  }
```
进入AppCompatActivity的setContentView：
```
 @Override
 public void setContentView(@LayoutRes int layoutResID) {
     getDelegate().setContentView(layoutResID);
 }
```
可以看到从这里开始，就和继承```Activity```的```setContentView```逻辑不同了。那么```getDelegate()```是什么呢？
从delegate我们可以参想这个应该是个代理模式。代理模式这边暂时不做研究。

****
#### 2.1、AppCompat的AppCompatDelegate 

跳转到getDelegate方法：
```
    @NonNull
    public AppCompatDelegate getDelegate() {
        if (mDelegate == null) {
            mDelegate = AppCompatDelegate.create(this, this);
        }
        return mDelegate;
    }
```
可以看到这里就是初始化了AppCompatDelegate。
跳转到AppCompatDelegate它有一行注释：
```
This class represents a delegate which you can use to extend AppCompat's support to any
```
这个注释的意思是：这个类表示一个代理，您可以使用它来扩展AppCompat的支持。那么AppCompatDelegate用来做什么呢？先继续往下看create方法的代码：
```
  private static AppCompatDelegate create(Context context, Window window,
            AppCompatCallback callback) {
        if (Build.VERSION.SDK_INT >= 24) {
            return new AppCompatDelegateImplN(context, window, callback);
        } else if (Build.VERSION.SDK_INT >= 23) {
            return new AppCompatDelegateImplV23(context, window, callback);
        } else if (Build.VERSION.SDK_INT >= 14) {
            return new AppCompatDelegateImplV14(context, window, callback);
        } else if (Build.VERSION.SDK_INT >= 11) {
            return new AppCompatDelegateImplV11(context, window, callback);
        } else {
            return new AppCompatDelegateImplV9(context, window, callback);
        }
    }
```
可以看到，这里根据版本判断返回了不同的```AppCompatDelegate```的实现。这些```AppCompatDelegateImpl```都继承自```AppCompatDelegateImplV9```。而且最终调用的是```AppCompatDelegateImplV9```的```setContentView```
直接跳转到```AppCompatDelegateImplV9```查看```setContentView```方法
```
    @Override
    public void setContentView(int resId) {
        ensureSubDecor();
        ViewGroup contentParent = (ViewGroup) mSubDecor.findViewById(android.R.id.content);
        contentParent.removeAllViews();
        LayoutInflater.from(mContext).inflate(resId, contentParent);
        mOriginalWindowCallback.onContentChanged();
    }
```
**AppCompatDelegate到底用来干什么的呢？**
AppCompatDelegate是一个可以放在任意Activity中，并且回调相应生命周期的类，在不使用AppCompatActivity的情况下,也可以得到一致的主题与颜色，用于兼容不同版本的控件。
具体看看这两篇博客：
http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0422/2774.html
http://www.jianshu.com/p/66fa79ec0109


****
#### 2.2 初始化SubDecorView

先看第一个方法：```ensureSubDecor()```
```
  private void ensureSubDecor() {
        if (!mSubDecorInstalled) {
            mSubDecor = createSubDecor();

            // If a title was set before we installed the decor, propagate it now
            CharSequence title = getTitle();
            if (!TextUtils.isEmpty(title)) {
                onTitleChanged(title);
            }
            applyFixedSizeWindow();
            onSubDecorInstalled(mSubDecor);
            mSubDecorInstalled = true;

            // Invalidate if the panel menu hasn't been created before this.
            // Panel menu invalidation is deferred avoiding application onCreateOptionsMenu
            // being called in the middle of onCreate or similar.
            // A pending invalidation will typically be resolved before the posted message
            // would run normally in order to satisfy instance state restoration.
            PanelFeatureState st = getPanelState(FEATURE_OPTIONS_PANEL, false);
            if (!isDestroyed() && (st == null || st.menu == null)) {
                invalidatePanelMenu(FEATURE_SUPPORT_ACTION_BAR);
            }
        }
      }
```
从以上代码，可以看到这里初始化了```mSubDecor```。
看下定义```mSubDecor```的地方：
```
    // true if we have installed a window sub-decor layout.
    private boolean mSubDecorInstalled;
    private ViewGroup mSubDecor;
```
这个注释的意思是：
```
  如果mSubDecorInstalled为true，则我们出事化了一个窗口的子装饰布局。
```
窗口的子装饰布局？ 先不管这是什么，先记住他。
 ```createSubDecor()```方法初始化了mSubDecor。跳转进去看下：
```
  private ViewGroup createSubDecor() {
        TypedArray a = mContext.obtainStyledAttributes(R.styleable.AppCompatTheme);
        //... ... 省略很多代码
        if (a.getBoolean(R.styleable.AppCompatTheme_windowActionModeOverlay, false)) {
            requestWindowFeature(FEATURE_ACTION_MODE_OVERLAY);
        }
        mIsFloating = a.getBoolean(R.styleable.AppCompatTheme_android_windowIsFloating, false);
        //... ... 省略很多代码
        // Now let's make sure that the Window has installed its decor by retrieving it
        mWindow.getDecorView();
}
```
好嘛，又见到了```TypedArray```，但是这次是取的```AppCompatTheme```。这回是取```AppCompatTheme```里面的style。这次不是赋值给```PhoneWindow```了，这次是赋值给```AppCompatDelegateImplBase```里面的成员变量。实际上也是```AppCompatDelegateImplV9```的成员变量，因为```AppCompatDelegateImplV9```继承自```AppCompatDelegateImplBase```。
不知道有开发者有没有注意到这个问题，如果Activity继承自AppCompatActivity，如果没有给这个Activity设置一个AppCompat的Theme。就会崩溃。原因就在这里了。
```
   if (!a.hasValue(R.styleable.AppCompatTheme_windowActionBar)) {
         a.recycle();
        throw new IllegalStateException(
                    "You need to use a Theme.AppCompat theme (or descendant) with this activity.");
    }

```
如果继承了AppCompatActivity没有设置Theme。抛出的异常一定是：
```
 "You need to use a Theme.AppCompat theme (or descendant) with this activity.
```
****
#### 2.2.1 初始化SubDecorView前，先初始化DecorView和mContentParent

初始化DecorView和mContentParent在前面已经了解了。 

接下来看下非常重要的代码：
```
    // Now let's make sure that the Window has installed its decor by retrieving it
    mWindow.getDecorView();
```
mWindow从AppCompatDelegateImplBase中的定义，可以知道它就是PhoneWindow。
那么跳转到PhoneWindow的getDecorView()方法。
```
    @Override
    public final View getDecorView() {
        if (mDecor == null || mForceDecorInstall) {
            installDecor();
        }
        return mDecor;
    }  
```
好嘛，```installDecor()``` 又是它。回顾已经讲过的installDecor()做了什么吧：
installDecor()方法的作用：
>初始化DecorView，出事化mContentParent并且加载一个LinearLayout。那么getDecorView就是初始化了DecorView。

继续往下看：
```
 final LayoutInflater inflater = LayoutInflater.from(mContext);
        ViewGroup subDecor = null;
        if (!mWindowNoTitle) {
            if (mIsFloating) {
                // If we're floating, inflate the dialog title decor
                subDecor = (ViewGroup) inflater.inflate(
                        R.layout.abc_dialog_title_material, null);
                // Floating windows can never have an action bar, reset the flags
                mHasActionBar = mOverlayActionBar = false;
            } else if (mHasActionBar) {
        }        
      }
```
这个逻辑又和前面初始化mContentParent并且加载一个LinearLayout的逻辑一样。

```
<android.support.v7.widget.FitWindowsLinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/action_bar_root"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:fitsSystemWindows="true">

    <android.support.v7.widget.ViewStubCompat
        android:id="@+id/action_mode_bar_stub"
        android:inflatedId="@+id/action_mode_bar"
        android:layout="@layout/abc_action_mode_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <include layout="@layout/abc_screen_content_include" />

</android.support.v7.widget.FitWindowsLinearLayout>
```
```
<merge xmlns:android="http://schemas.android.com/apk/res/android">

    <android.support.v7.widget.ContentFrameLayout
            android:id="@id/action_bar_activity_content"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:foregroundGravity="fill_horizontal|top"
            android:foreground="?android:attr/windowContentOverlay" />

</merge>
```
可以看到这个和已经讲过的初始化mContentParent时加载LinearLayout差不多。请记住ContentFrameLayout这个View的id： ``` android:id="@id/action_bar_activity_content"``` 继续看代码：
****
#### 2.2.2 ContentFrameLayout 和mContentParent的相互替换

```
 final ContentFrameLayout contentView = (ContentFrameLayout) subDecor.findViewById(
                R.id.action_bar_activity_content);

        final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content);
        if (windowContentView != null) {
            // There might be Views already added to the Window's content view so we need to
            // migrate them to our content view
            while (windowContentView.getChildCount() > 0) {
                final View child = windowContentView.getChildAt(0);
                windowContentView.removeViewAt(0);
                contentView.addView(child);
            }

            // Change our content FrameLayout to use the android.R.id.content id.
            // Useful for fragments.
            windowContentView.setId(View.NO_ID);
            contentView.setId(android.R.id.content);

            // The decorContent may have a foreground drawable set (windowContentOverlay).
            // Remove this as we handle it ourselves
            if (windowContentView instanceof FrameLayout) {
                ((FrameLayout) windowContentView).setForeground(null);
            }
        }

        // Now set the Window's content view with the decor
        mWindow.setContentView(subDecor);
```
这段代码的逻辑：
首先从```subDecor```中通过```findViewById```的方法获取到```ContentFrameLayout```。
然后通过```PhoneWindow```的```findViewById```方法获取到了```DecorView```中的```LinearLayout```中的```FrameLayout```。
然后遍历```DecorView```中的```FrameLayout```将它的子View添加到```ContentFrameLayout```。
这个时候 ```DecorView```中的```FrameLayout```就没有子VIew了。
```
// Change our content FrameLayout to use the android.R.id.content id.
            // Useful for fragments.
            windowContentView.setId(View.NO_ID);
            contentView.setId(android.R.id.content);

```
然后把```DecorView```中的```FrameLayout```的Id设为NO_ID
把```ContentFrameLayout```设为```android.R.id.content```。
这里就是 ```ContentFrameLayout```替换了```DecorView```中的```FrameLayout```的过程。

```
   mWindow.setContentView(subDecor);
```
然后调用```mWindow.setContentView(subDecor);```把SubDecorVIew加载到PhoneWIndow的mContentParent中去。就这样继承AppCompatActivity的Activity调用setContentView就这样完成了。

前面提到过SubDecorVIew是窗口的子装饰布局，这里确实将SubDecorVIew加载到了DecorView中。

****
#### 2.3 画图来表示继承AppCompatActivity的setContentView的流程。
继承AppCompatActivity的```setContentView()```最后调用的是```createSubDecor()```。Acitvity加载布局的逻辑都在这个方法中：
**第一步：初始化DecorView和mContentParent**
代码为：
```
        // Now let's make sure that the Window has installed its decor by retrieving it
        mWindow.getDecorView();
```
![初始化DecorView和mContentParent](http://upload-images.jianshu.io/upload_images/1930161-1e14f419d599ba67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**第二步：初始化SubDecorView**
代码为：


```
  final LayoutInflater inflater = LayoutInflater.from(mContext);
        ViewGroup subDecor = null;
// ... ... 省略若干代码
 subDecor = (ViewGroup) inflater.inflate(R.layout.abc_screen_simple, null);
```
![初始化SubDecorView](http://upload-images.jianshu.io/upload_images/1930161-a612a29811c330b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**第三步：SubDecorView和DecorView中的mContentParent替换的过程**
代码为：
```
 final ContentFrameLayout contentView = (ContentFrameLayout) subDecor.findViewById(
                R.id.action_bar_activity_content);

        final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content);
        if (windowContentView != null) {
            // There might be Views already added to the Window's content view so we need to
            // migrate them to our content view
            while (windowContentView.getChildCount() > 0) {
                final View child = windowContentView.getChildAt(0);
                windowContentView.removeViewAt(0);
                contentView.addView(child);
            }

            // Change our content FrameLayout to use the android.R.id.content id.
            // Useful for fragments.
            windowContentView.setId(View.NO_ID);
            contentView.setId(android.R.id.content);

            // The decorContent may have a foreground drawable set (windowContentOverlay).
            // Remove this as we handle it ourselves
            if (windowContentView instanceof FrameLayout) {
                ((FrameLayout) windowContentView).setForeground(null);
            }
        }

```
![SubDecorView和DecorView中的mContentParent替换的过程](http://upload-images.jianshu.io/upload_images/1930161-230766faac8c67ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**第四步：将SubDecorView加载到DecorView中去**
代码为：
```
        // Now set the Window's content view with the decor
        mWindow.setContentView(subDecor);
```
![将SubDecorView加载到DecorView中去](http://upload-images.jianshu.io/upload_images/1930161-5823a27c3041d530.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




**第五步：将资源文件中的布局添加到ContentFrameLayout中**
代码为：
```
 @Override
    public void setContentView(int resId) {
        ensureSubDecor();
        ViewGroup contentParent = (ViewGroup) mSubDecor.findViewById(android.R.id.content);
        contentParent.removeAllViews();
        LayoutInflater.from(mContext).inflate(resId, contentParent);
        mOriginalWindowCallback.onContentChanged();
    }
```
最后继承AppCompatActivity的布局结构：
![继承AppCompatActivity的布局结构](http://upload-images.jianshu.io/upload_images/1930161-92a2af88094f0c41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看到这么多层的布局不要惊讶，事实就是这样。

#### 3、总结
以上就是继承Activity和继承AppCompatActivity的逻辑和布局结构。如果已经看懵逼了就看下画了图的内容吧，那是总结出来的精华。


#### 有些话写在最后
如果最后有人看完了这篇博客并且大概了解博客讲的东西，真的非常感谢。看源码对于现在的我来说真的是一个非常痛苦的过程。为了尽量写好这篇文章源码看了一遍又一遍。写完之后确实是非常轻松的，因为确实学到了不少干货。















