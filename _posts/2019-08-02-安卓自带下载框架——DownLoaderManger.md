---
layout:     post
title:      安卓自带下载框架——DownLoaderManger
subtitle:   学好这个东西，下载啥的都不是问题。
date:       2019-08-03
author:     陈再峰
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
    - DownLoaderManger
---

已经超过大半年没有写博客了，这大半年还是学到了很多的东西，但是一直缺少总结，写博客的目的无非就以下几个：1、对自己学习的技术做好总结。 2、分享自己学到的东西，可能会给其他开发的伙伴带来帮助。3、给分享技术，开源带来自己的一份微薄之力。

DownLoaderManger这个老早以前就想发表一篇博客了，刚刚接触的时候，确实非常方便而且很简单。这篇博客主要涉及到封装DownLoaderManger下载器，以及FileProvider对androidX的一些兼容问题。

#### 1、主要逻辑
![主要逻辑](https://upload-images.jianshu.io/upload_images/1930161-30177700e1528ea8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 2、需要的权限
完成一个apk从下载到安装需要的权限如下：
* 需要访问网络，添加网络权限。
* 需要下载文件和安装apk，添加读写文件权限。
* 需要安装apk，添加请求安装应用的权限。

```
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />
```
#### 3、安卓自带的DownloadManger
![DownloadManger](https://upload-images.jianshu.io/upload_images/1930161-f4f18f8bd7625fe8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
安卓自带的DownLoaderManger是安卓系统提供的一个下载器，它是一个系统的服务。获取的方式为：
```
var downloadManager: DownloadManager = context.getSystemService(Context.DOWNLOAD_SERVICE) as DownloadManager
```
DownloadManager有一个Request来构建下载的一些参数和notification的一些样式等等。最后通过DownloadManager的```DownloadManager.enqueue()```方法启动下载。以下是源码中的部分代码（具体源码地址在文章最后）。
```
           // 真正的下载逻辑,Request用来构建下载所使用的参数。
            val request = DownloadManager.Request(Uri.parse(url))
            request.setTitle(title)
            request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE or DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED)
            request.setMimeType("application/vnd.android.package-archive")
            if (TextUtils.isEmpty(title)) {
                request.setDescription(fileName)
            } else {
                request.setDescription(title)
            }

            try {
                if (DownLoaderManager.downloadManager == null) {
                    // 可能部分机型不支持这个DownloaderManger ，这边可以使用自定义的下载框架
                    return
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }


            request.setDestinationUri(Uri.fromFile(file))
            if (DownLoaderManager.downloadManager != null) {
                downloadId = DownLoaderManager.downloadManager!!.enqueue(request)
           }
```
如果代码执行成功则会在通知栏中展示。
 ![下载的状态](https://upload-images.jianshu.io/upload_images/1930161-147889c58237a8bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4、下载完成的监听
DownloaderManger文件下载完成之后是通过广播监听的，然后通过downLoadId来获取文件的Uri，最后通过Uri安装软件。   
广播注册的方式有两种，这里介绍在AndroidMainfest中的这种：
```
        <receiver android:name=".downloader.DownLoaderBroadcastReceiver">
            <intent-filter>
                <action android:name="android.intent.action.DOWNLOAD_COMPLETE"/>
            </intent-filter>
        </receiver>
```
```
class DownLoaderBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
   
    }
}
```
#### 5、File Provider在7.0和androidX上的区别

File Provider是在安卓7.0的新特性，但是在x上又有不同的区别，实际上修改起来也非常简单，只需要修改一个参数即可。    
7.0：
```
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="${applicationId}.fileProvider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
```
androidX:      
```
        <provider
                android:name="androidx.core.content.FileProvider"
                android:authorities="${applicationId}.fileProvider"
                android:exported="false"
                android:grantUriPermissions="true">
            <meta-data
                    android:name="android.support.FILE_PROVIDER_PATHS"
                    android:resource="@xml/file_paths"/>
        </provider>
```
就name不一样，其他的都一样。

#### 6、源码

注意：    
* 这边用的是androidX，这边的file provider有所区别。
* 语言使用的是Kotlin（Kotlin已经是官方语言哦，建议安卓开发都用它）

[https://github.com/AxeChen/DownLoaderManagerDemo](https://github.com/AxeChen/DownLoaderManagerDemo)
如果对你有帮助，点个赞再走哦。：）

> 本文章在简书上发布，其他平台使用请标明来源和作者，谢谢！否则都为盗版。

