---
layout:     post
title:      Android-Material-Design：BottomSheetBehavior
subtitle:   Google的MD设计控件，赶紧学起来！
date:       2017-09-17
author:     陈再峰
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
    - Material Design
---

BottomSheet是design23.3推出的底部动作条，google原生自带的软件就有这种效果。
效果图：
![Google联系人](http://upload-images.jianshu.io/upload_images/1930161-553308f727d37c38.gif?imageMogr2/auto-orient/strip)
我自己做的效果：
![效果图](http://upload-images.jianshu.io/upload_images/1930161-3b014a94c8d17ec5.gif?imageMogr2/auto-orient/strip)

示例代码中的关于BottomSheet的实际有三个：

> 1、BottomSheetBehavior
2、BottomSheetDialogFragment
3、BottomSheetDialog

**如果要实现BottomSheet的效果。第一步保证导入的design依赖在23.3或以上。**
```
 compile 'com.android.support:design:26.0.0-alpha1'
```

### 1、BottomSheetBehavior
* 在xml文件中加入``` app:layout_behavior="@string/bottom_sheet_behavior"```然后通过控制BottomSheetBehavior的状态来控制展示。

```
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.mg.axe.bottomsheetsimple.MainActivity">

    <LinearLayout
        android:id="@+id/llBottom"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:behavior_hideable="false"
        app:behavior_peekHeight="80dp"
        app:layout_behavior="@string/bottom_sheet_behavior">

       <!--自己定义的布局-->

    </LinearLayout>
</android.support.design.widget.CoordinatorLayout>
```
代码控制：
* 先初始化BottomSheetBehavior
BottomSheetBehavior.from(设置了app:layout_behavior="@string/bottom_sheet_behavior"的view)
```
BottomSheetBehavior bottomSheetBehavior = BottomSheetBehavior.from(linearLayout);
```
* 控制展开和收缩：
```
  if (bottomSheetBehavior.getState() == BottomSheetBehavior.STATE_EXPANDED) {
            bottomSheetBehavior.setState(BottomSheetBehavior.STATE_COLLAPSED);
  } else {
            bottomSheetBehavior.setState(BottomSheetBehavior.STATE_EXPANDED);
  }
```
#### 1.1、常用属性和状态
* 常用属性 

|属性|作用|
|:-------:|:-------:|
| app:behavior_peekHeight | BottomSheet收缩时展示的高度 |
|app:behavior_hideable|滑动是否可隐藏（一般是向下滑动）|
* 常用状态

|状态|含义|
|:-------:|:-------:|
|STATE_COLLAPSED|收缩状态|
|STATE_EXPANDED|展开状态|
|STATE_DRAGGING|正在拖动状态|
|STATE_HIDDEN|隐藏状态|

**注意：以上介绍的属性和状态并不完善。可去看官方文档**
https://developer.android.com/reference/android/support/design/widget/BottomSheetBehavior.html
(记得翻墙，文明上网)

* 状态的回调
```
  bottomSheetBehavior.setBottomSheetCallback(new BottomSheetBehavior.BottomSheetCallback() {
            @Override
            public void onStateChanged(@NonNull View bottomSheet, int newState) {
                // newState 回调的状态
            }

            @Override
            public void onSlide(@NonNull View bottomSheet, float slideOffset) {

            }
        });
```
### 2、BottomSheetDialogFragment和BottomSheetDialog  
BottomSheetDialogFragment和BottomSheetDialog效果都一样。都是在底部向上弹出Dialog。
#### 2.1、BottomSheetDialogFragment
BottomSheetDialogFragment继承自DialogFragment和DialogFragment的用法一摸一样。
DialogFragment：http://blog.csdn.net/lmj623565791/article/details/37815413/
唯一注意的地方是，弹出Dialog之后状态栏变成黑色。
解决方案：http://blog.csdn.net/maosidiaoxian/article/details/52288597

自定义BottomSheetDialogFragment代码：
```
public class MyBottomSheetDialogFragment extends BottomSheetDialogFragment {

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        // 自定义布局
        View view = inflater.inflate(R.layout.dialog_bottom, container, false);
        TextView gotit = (TextView) view.findViewById(R.id.gotIt);
        gotit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dismiss();
            }
        });

        return view;
    }

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        // 解决dialog黑屏问题
        Dialog dialog = super.onCreateDialog(savedInstanceState);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            dialog.getWindow().addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
        }
        return dialog;
    }
}
```
使用方法：
```
MyBottomSheetDialogFragment dialogFragment = new MyBottomSheetDialogFragment();
dialogFragment.show(getSupportFragmentManager(), "MyBottomSheetDialogFragment");
```

#### 2.2 BottomSheetDialog
BottomSheetDialog继承自AppCompatDialog。使用方法也非常简单：
```
  BottomSheetDialog bottomSheetDialog = new BottomSheetDialog(this);
  bottomSheetDialog.setContentView(R.layout.dialog_bottom);
  bottomSheetDialog.show();
```
**注意：**这里只是简单的展示，如果复杂的逻辑可能需要自定义BottomSheetDialog

### 3、最后
源码地址:https://github.com/AxeChen/MaterialDesignSimple/tree/master/MaterialDesignSimple/bottomsheetsimple
其他关于Material Design博客。
**Android Material Design：常用控件学习笔记**
http://www.jianshu.com/p/776cc6329fff 
**Android Material Design：使用Palette优化界面色彩搭配**
http://www.jianshu.com/p/dfa9aac6143d
