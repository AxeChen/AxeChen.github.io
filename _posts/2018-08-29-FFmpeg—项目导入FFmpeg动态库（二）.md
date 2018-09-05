---
layout:     post
title:      FFmpeg—项目导入FFmpeg动态库（二）
subtitle:   FFmpeg系列
date:       2018-08-29
author:     陈再峰
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
	- FFmpeg
---

AndroidStudio导入FFmpeg动态库和一般的NDK开发导入动态库一样。首先我们准备好动态库。
那么FFmpeg的动态库哪里来呢？


可以看看我写的博客：[NDK开发——Linux编译FFmpeg](https://www.jianshu.com/p/e4b862e6fd7e)这篇博客会告诉你怎么编译动态库。            
当然也有编译好的现成的so库，百度网盘：链接：https://pan.baidu.com/s/1AjOJYbh6dfWgOOFzIxM9Kw 密码：ikr8        


#### 1、新建NDK项目
新建NDK项目和建一般的项目差不多，有两个点需要注意：
* 新建项目的时候将 ```include C++ support```勾选上。     
![include C++](https://upload-images.jianshu.io/upload_images/1930161-e8b1b755791b3dff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 添加c++的异常处理支持（可选）   
![TIM截图20180829105535.png](https://upload-images.jianshu.io/upload_images/1930161-014c0e200d3b62fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一路点到finish，然后就会构建NDK项目了，第一次构建NDK项目时间有点长，这个时候你可以掏出手机打一把王者农药。


#### 2、导入动态库
导入动态库的方式有两种哦，我这里只介绍一种。
* 第一种将动态库放到jniLibs文件夹中。（不介绍）
* 将动态库导入到libs中   
 ![项目结构](https://upload-images.jianshu.io/upload_images/1930161-6fe69392314da9a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
记住这个文件夹名称：armeabi-v7a  这个是必须这样命名的，因为我编译的就是arm cpu架构的动态库，用armeabi命名也不行哦，会报如下错误：
```
ABIs [armeabi] are not supported for platform. Supported ABIs are [armeabi-v7a, arm64-v8a, x86, x86_64].
```
然后需要在app下的build.gradle下添加如下配置：
```
externalNativeBuild {
            cmake {
                cppFlags "-frtti -fexceptions"
                abiFilters 'armeabi-v7a'
            }
        }

        sourceSets {
            main {
                jniLibs.srcDirs = ['libs']
            }
        }
```

#### 3、配置CMakeList.txt
将动态库导入之后，还需要配置好CMakeList.txt将动态库链接进去。
```
cmake_minimum_required(VERSION 3.4.1)

add_library( native-lib
             SHARED
             src/main/cpp/native-lib.cpp )


find_library( log-lib
              log )

# 声明一个DIR的变量获取libs的路径
set(DIR ../../../../libs)

# 引入头文件
include_directories(libs/include)

# 引入所有的so动态库
add_library(avcodec-56
            SHARED
            IMPORTED)
set_target_properties(avcodec-56
                      PROPERTIES IMPORTED_LOCATION
                      ${DIR}/armeabi-v7a/libavcodec-56.so)

add_library(avdevice-56
            SHARED
            IMPORTED)
set_target_properties(avdevice-56
                      PROPERTIES IMPORTED_LOCATION
                      ${DIR}/armeabi-v7a/libavdevice-56.so)

add_library(avformat-56
            SHARED
            IMPORTED)
set_target_properties(avformat-56
                      PROPERTIES IMPORTED_LOCATION
                      ${DIR}/armeabi-v7a/libavformat-56.so)

add_library(avutil-54
            SHARED
            IMPORTED)
set_target_properties(avutil-54
                      PROPERTIES IMPORTED_LOCATION
                      ${DIR}/armeabi-v7a/libavutil-54.so)

add_library(postproc-53
            SHARED
            IMPORTED)
set_target_properties(postproc-53
                      PROPERTIES IMPORTED_LOCATION
                      ${DIR}/armeabi-v7a/libpostproc-53.so)


add_library(swresample-1
             SHARED
             IMPORTED)
set_target_properties(swresample-1
                       PROPERTIES IMPORTED_LOCATION
                       ${DIR}/armeabi-v7a/libswresample-1.so)

add_library(swscale-3
              SHARED
              IMPORTED)
set_target_properties(swscale-3
                        PROPERTIES IMPORTED_LOCATION
                        ${DIR}/armeabi-v7a/libswscale-3.so)


add_library(avfilter-5
              SHARED
              IMPORTED)
set_target_properties(avfilter-5
                        PROPERTIES IMPORTED_LOCATION
                        ${DIR}/armeabi-v7a/libavfilter-5.so)


# 链接动态库

target_link_libraries( native-lib
                      avcodec-56
                      avdevice-56
                      avformat-56
                      avutil-54
                      postproc-53
                      swresample-1
                      swscale-3
                      avfilter-5
                      ${log-lib} )
```

#### 4、测试编译是否能通过
在自动生成的MainActivity中加载这些动态库，然后运行下，看看能不能跑起来吧。不过估计还没到这一步，上面的几步点一下Sync Now同步代码编译不通过也是常有的事。
```
    static {
        System.loadLibrary("avcodec-56");
        System.loadLibrary("avdevice-56");
        System.loadLibrary("avfilter-5");
        System.loadLibrary("avformat-56");
        System.loadLibrary("avutil-54");
        System.loadLibrary("postproc-53");
        System.loadLibrary("swresample-1");
        System.loadLibrary("swscale-3");
        System.loadLibrary("native-lib");
    }
```
如果编译运行成功，就可以开始使用FFmpeg提供的api做一些事情啦。

#### 5、需要了解的一些东西
如果要熟练的导入动态库到项目中，CMakeList.txt的语法需要了解，还有就是两种不同的导入平台的so库（比如这里只导入了arm下的so库），导入平台库的不同配置方式。



代码在这里：[https://github.com/AxeChen/FFmpegStudy](https://github.com/AxeChen/FFmpegStudy)
