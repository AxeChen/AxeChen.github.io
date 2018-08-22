---
layout:     post
title:      获取系统顶部状态栏(Status Bar)与底部导航栏(Navigation Bar)的高度
date:       2016-10-26
author:     陈再峰
tags:
    - Android
---   

Android一些设备都有上下两条bar，我们可以获取这些bar的信息。
原文地址：http://www.cnblogs.com/rossoneri/p/4142962.html
>获取顶部status bar 高度

```
private int getStatusBarHeight() {
    Resources resources = mActivity.getResources();
    int resourceId = resources.getIdentifier("status_bar_height", "dimen","android");
    int height = resources.getDimensionPixelSize(resourceId);
    Log.v("dbw", "Status height:" + height);
    return height;
}
```


>获取底部 navigation bar 高度

```
private int getNavigationBarHeight() {
    Resources resources = mActivity.getResources();
    int resourceId = resources.getIdentifier("navigation_bar_height","dimen", "android");
    int height = resources.getDimensionPixelSize(resourceId);
    Log.v("dbw", "Navi height:" + height);
    return height;
}
```
