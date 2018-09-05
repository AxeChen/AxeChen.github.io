---
layout:     post
title:      FFmpeg—Linux编译FFmpeg动态库（一）
subtitle:   FFmpeg系列
date:       2018-08-28
author:     陈再峰
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
    - NDK
	- FFmpeg
---

如果需要学习FFmpeg，则需要学会编译FFmpeg，在安卓开发中，一般是将FFmpeg的源代码编译成动态库供安卓项目调用。这里编译FFmpeg可以用Linux和Mac，这里只介绍Linux的编译FFmpeg的情况。
#### 1、在这之前需要做的准备工作：
* 准备一个Linux系统
* 了解Linux的shell脚本
* 了解Linux的gcc编译
* 了解Linux的一些常用命令

**获得Linux系统的方式**
* 安装虚拟机
* 购买Linux云服务器
* 电脑上装一个Linux


#### 2、安装NDK编译环境
编译FFmpeg需要用到NDK的编译环境，所以在Linux中编译FFmpeg第一步需要先将NDK环境搞起来。先要下载NDK的安装包，有两种方式可以下载：

##### 2.1、下载NDK
* 直接在网站下载
NDK下载地址：[https://developer.android.google.cn/ndk/downloads](https://developer.android.google.cn/ndk/downloads) 下载完之后是一个zip的文件，然后把这个文件传到Linux系统中去。

* 直接使用Linux的wget命令下载
```
wget https://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip
```
```
[root@VM_0_16_centos ndk]# wget https://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip
--2018-08-28 11:39:51--  https://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip
Resolving dl.google.com (dl.google.com)... 203.208.41.34, 203.208.41.32, 203.208.41.38, ...
Connecting to dl.google.com (dl.google.com)|203.208.41.34|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1110915721 (1.0G) [application/zip]
Saving to: ‘android-ndk-r10e-linux-x86_64.zip’

```
下载完之后是一个zip格式的文件，需要用到Linux种的```unzip```解压zip的命令
```
unzip android-ndk-r10e-linux-x86_64.zip
```
如果提示权限不足，就使用以下命令：
```
chmod 777 android-ndk-r10e-linux-x86_64.zip
```
##### 2.2、设置NDK编译环境

如果解压成功了，接下来就是设置环境变量了。当Linux系统已启动就会去加载这些配置文件，这个和Windows设置环境变量的原理是一样的。设置环境变量的方式有两种：
**这里都会修改系统配置文件，请先切换成root去操作**
* 第一种，修改/etc/profile。
```
vim /etc/profile
```
然后早profile中添加如下代码：    
注意你自己的具体路径哦，```/home/axe/```是我自己系统中的位置。
```
NDKROOT=/home/axe/ndk/android-ndk-r10e 
PATH=$PATH:$NDKROOT
export PATH NDKROOT
```
然后重启Linux系统。
* 第二种，修改.bashrc文件
```
vim ~/.bashrc
```
然后早.bashrc中添加如下代码（写在fi后面）：    
注意你自己的具体路径哦，```/home/axe/```是我自己系统中的位置。
```
export NDKROOT=/home/axechen/ndk/android-ndk-r10e
export PATH=$NDKROOT:$PATH
```
保存之后，更新资源文件命令：
```
source ~/.bashrc
```
（两种方法配置完成之后）然后输入如下命令看看是否配置成功：
```
ndk-build -v
```
如果出现类似如下日志：
```
GNU Make 3.81
Copyright (C) 2006  Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.
```
那ndk编译环境搭好了：）。

**我这里下载的是android-ndk-r10e版本，也可下载其他的版本。**

#### 3、FFmpeg的下载和解压
##### 3.1、下载FFmpeg
下载FFmpeg和下载NDK是一样的，两种方式下载，一种是网站直接下载压缩包，另一种是通过命令行下载。我这里下载的是2.6.9的稳定版本，当然也可去下载其他的版本。
ffmpeg的下载地址：[http://ffmpeg.org/releases/](http://ffmpeg.org/releases/)

我这里用命令行的方式下载：
```
 wget  http://ffmpeg.org/releases/ffmpeg-2.6.9.tar.gz
```
然后就开始下载ffmpeg：
```
Resolving ffmpeg.org (ffmpeg.org)... 79.124.17.100
Connecting to ffmpeg.org (ffmpeg.org)|79.124.17.100|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9744514 (9.3M) [application/x-gzip]
Saving to: ‘ffmpeg-2.6.9.tar.gz’
```

##### 3.2、解压FFmpeg
下载完成之后是一个```ffmpeg-2.6.9.tar.gz```文件，将这个文件解压：
```
tar -xzf ffmpeg-2.6.9.tar.gz
```
给解压后的文件赋予权限：
```
 chmod 777 ffmpeg-2.6.9
```
看下ffmpeg里面的东西：这个里面都是ffmpeg的源文件，当然不需要你花时间去弄懂这些东西，里面有个configure文件是比较重要的。
```
[root@VM_0_16_centos ffmpeg]# cd ffmpeg-2.6.9
[root@VM_0_16_centos ffmpeg-2.6.9]# ls
arch.mak                COPYING.GPLv2     ffmpeg.h           INSTALL.md     libswresample  tests
Changelog               COPYING.GPLv3     ffmpeg_opt.c       libavcodec     libswscale     tools
cmdutils.c              COPYING.LGPLv2.1  ffmpeg_vda.c       libavdevice    LICENSE.md     VERSION
cmdutils_common_opts.h  COPYING.LGPLv3    ffmpeg_vdpau.c     libavfilter    MAINTAINERS    version.sh
cmdutils.h              CREDITS           ffplay.c           libavformat    Makefile
cmdutils_opencl.c       doc               ffprobe.c          libavresample  presets
common.mak              ffmpeg.c          ffserver.c         libavutil      README.md
compat                  ffmpeg_dxva2.c    ffserver_config.c  libpostproc    RELEASE
configure               ffmpeg_filter.c   ffserver_config.h  library.mak    RELEASE_NOTES
```
看下configure里面的东西：
```
#!/bin/sh
#
# FFmpeg configure script
#
# Copyright (c) 2000-2002 Fabrice Bellard
# Copyright (c) 2005-2008 Diego Biurrun
# Copyright (c) 2005-2008 Mans Rullgard
#

# Prevent locale nonsense from breaking basic text processing.
LC_ALL=C
export LC_ALL

# make sure we are running under a compatible shell
# try to make this part work with most shells

try_exec(){
    echo "Trying shell $1"
    type "$1" > /dev/null 2>&1 && exec "$@"
}

unset foo
(: ${foo%%bar}) 2> /dev/null
E1="$?"
# ... ...
```
configure是shell脚本，可以选择你要编译的类库。FFmpeg就主要就是通过这个文件的配置来编译。
搞不懂这个文件没有关系，选择一个网上已经写好的shell脚本去编译会使编译边得更简单。这边有个修改了configure的文件，会编译出windows直接可以使用的so库。
修改后的configure和编译的shell脚本下载地址如下：   
链接：[https://pan.baidu.com/s/1xQyregaXK5dG-RAxxZkruw](https://pan.baidu.com/s/1xQyregaXK5dG-RAxxZkruw) 密码：jgiz
然后将这两个文件放在configure的文件夹下，这边会替换掉configure：   
**注意：**如果需要修改这两个文件，请上传到Linux系统之后，在Linux系统修改，否则因为编码问题会导致无法编译通过。
![替换文件](https://upload-images.jianshu.io/upload_images/1930161-3714427675fa481c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 3.3、编译FFmpeg
在编译FFmpeg时请先修改一下build_android.sh里面的参数配置：
```
root@VM_0_16_centos ffmpeg-2.6.9]# vim build_android.sh

#!/bin/bash
make clean
export NDK=/root/ndk/android-ndk-r10e
export SYSROOT=$NDK/platforms/android-9/arch-arm
export TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64
export CPU=arm
export PREFIX=$(pwd)/android/$CPU
export ADDI_CFLAGS="-marm"

./configure --target-os=linux \
--prefix=$PREFIX --arch=arm \
--disable-doc \
--enable-shared \
--disable-static \
--disable-yasm \
--disable-symver \
--enable-gpl \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
--disable-doc \
--disable-symver \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install

```
NDK、SYSROOT、TOOLCHAIN 请按照安装的的NDK目录修改正确。      
CPU参数表明编译的cpu类型；PREFIX表明输出的路径；
修改完毕之后就保存，如果有错误时无法编译成功的。

```
./configure --target-os=linux \
```
这个就是调用了configure脚本，然后下面的```enable```和```disable```就决定需要编译的文件。这里配置的就只是编译一些音视频解析的动态库。

```
./build_android.sh
```
执行这断命令之后，如果配置都正确就会开始编译，编译的时间可能会有点长，这个时候你可以掏出手机吃把鸡。
![正在编译](https://upload-images.jianshu.io/upload_images/1930161-48cc85f2388255e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

编译成功之后，就能在build_android.sh配置的输出路径中找到编译的so和头文件了
```
[root@VM_0_16_centos ffmpeg-2.6.9]# cd android
[root@VM_0_16_centos android]# ls
arm
[root@VM_0_16_centos android]# cd arm
[root@VM_0_16_centos arm]# ls
include  lib
[root@VM_0_16_centos arm]# 

```
lib 中是so动态库文件，   include为.h的头文件。
![image.png](https://upload-images.jianshu.io/upload_images/1930161-964a6dd5d7b78bff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1930161-e0ed9437384a2117.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后将```xxx-数字.so``` 和整个 include文件夹保存起来，准备给项目导入。 
