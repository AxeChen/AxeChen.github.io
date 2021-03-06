---
layout:     post
title:      FFmpeg—SurfaceView播放视频（三）
subtitle:   FFmpeg视频解析哦。
date:       2018-12-11
author:     陈再峰
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Android
    - FFmpeg
---

隔上个FFmpeg博客已经相隔很远了，主要是FFmpeg确实太难了，像我这种英文蹩脚，而且c/c++基础比较薄弱的人来说，学起来非常吃力。以下是FFmpeg解析视频将其在SurfaceView上播放视频的代码。

#### 1、SurfaceView播放视频大概流程
MP4，AVI等为一种h264的视频编码格式，如果需要在SurfaceView中展示需要将其转化为RGB格式并一帧一帧的画到SurfaceView中。
关于视频编码格式的拓展：[https://blog.csdn.net/houxiaoni01/article/details/78812485/](https://blog.csdn.net/houxiaoni01/article/details/78812485/
)
![流程](https://upload-images.jianshu.io/upload_images/1930161-0614c0186a23312d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个是一个大概的流程，详细流程是非常复杂的。

#### 2、常用的一些api和方法
FFmpeg去解码转换视频是一个复杂的过程，在Android中是在c层完成的，所以需要具备一定的c/c++编码能力。而且里面用到的方法和struct也是非常多。
* av_register_all() 初始化ffmpeg，基本用到ffmpeg第一句代码就是这个。
* AVFormatContext 视频格式上下文，视频流可以通过这个struct来获取。
* avformat_open_input() 打开一个输入流并读取标题。
* avformat_find_stream_info() 读取一个媒体文件的数据包以获取流信息失败
* AVCodecContext 编解码上下文，用于解码MP4。
* avcodec_open2() 初始化AVCodecContext。
* AVCodec 解码器，通过avCodecContext获得。
* SwsContext 格式转换，用于配置转换的格式，宽高等其他属性。
* AVPacket 数据包，可以理解为一帧包含的所有信息。
* AVFrame 视频/音频的原始信息。
* av_read_frame() 读取视频流的信息到AVPacket中
* ANativeWindow 从SurfaceView中获取的原生窗体属性。
* avcodec_decode_video2() 将AVPacket中的信息解析到AVFrame中

**api的初始化流程大概如下：**
![API的初始化流程](https://upload-images.jianshu.io/upload_images/1930161-9749d12089cc608a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果你看到这里，是不是感觉已经蒙圈了？没关系看完代码会更加蒙圈。况且我写完代码之后也是有些蒙圈的。

#### 3、具体代码
为了使这些代码能够看懂，我还用了“有道翻译”将FFmpeg中的注释做了一些翻译。如果要我不看笔记去再写一份代码出来，绝对是做不到的。
```
extern "C"
JNIEXPORT void JNICALL
Java_com_app_axe_ffmpegstudy_playVideo_VideoView_playVideo(JNIEnv *env, jobject instance,
                                                           jstring path_, jobject surface) {
    const char *path = env->GetStringUTFChars(path_, 0);
    // 初始化ffmpeg
    av_register_all();

    // 获取AVFormatContext ,这个类存储这文件的音频和视频流信息
    AVFormatContext *avFormatContext = avformat_alloc_context();

    // 打开一个输入流并读取到AVFormatContext中，如果路径不对或者其他情况会返回-1 .返回-1则表示读文件出错
    int code = avformat_open_input(&avFormatContext, path, NULL, NULL);
    if (code < 0) {
        LOGE("打开一个输入流并读取标题失败");
        return;
    }

    // 读取一个媒体文件的数据包以获取流信息  如果返回0则获取成功，-1则获取失败
    int findCode = avformat_find_stream_info(avFormatContext, NULL);
    if (findCode < 0) {
        LOGE("读取一个媒体文件的数据包以获取流信息失败");
        return;
    }

    int video_stream_id = -1;
    // 遍历文件流，找到里面的视频流位置(因为文件可能有多个流（视频流，音频流）等等 )
    // nb_streams 代表流的个数
    for (int i = 0; i < avFormatContext->nb_streams; ++i) {
        // AVMEDIA_TYPE_VIDEO 来自 AVMediaType ，里面定义了多个类型
        if (avFormatContext->streams[i]->codec->codec_type == AVMEDIA_TYPE_VIDEO) {
            video_stream_id = i;
            break;
        }
    }

    // Codec context associated with this stream. Allocated and freed by  libavformat.
    // 与此流相关的编解码上下文。由libavformat分配和释放。
    // 获取到编解码上下文。根据视频流id获取
    AVCodecContext *avCodecContext = avFormatContext->streams[video_stream_id]->codec;

    // Find a registered decoder with a matching codec ID.
    // 找到一个带有匹配的编解码器ID的注册解码器。
    // 获取解码器
    AVCodec *avCodec = avcodec_find_decoder(avCodecContext->codec_id);

    // 打开解码器
    // Initialize the AVCodecContext to use the given AVCodec. Prior to using this
    // function the context has to be allocated with avcodec_alloc_context3().
    // 初始化AVCodecContext以使用给定的AVCodec。之前使用这个
    // 必须使用avcodecalloc context3（）分配上下文。
    int codecOpenCode = avcodec_open2(avCodecContext, avCodec, NULL);
    if (codecOpenCode < 0) {
        LOGE("初始化AVCodec失败");
        return;
    }
    // 格式转换关键类
    // 首先这是mp4 如果需要解析成 yuv 需要用到 SwsContext
    // 构造函数传入的参数为 原视频的宽高、像素格式、目标的宽高这里也取原视频的宽高（可以修改参数）
    SwsContext *swsContext = sws_getContext(avCodecContext->width, avCodecContext->height,
                                            avCodecContext->pix_fmt,
                                            avCodecContext->width, avCodecContext->height,
                                            AV_PIX_FMT_RGBA, SWS_BICUBIC, NULL, NULL, NULL);
    // 解析的每一帧都是 AVPacket
    // 初始化AVPacket对象
    AVPacket *avPacket = (AVPacket *) av_malloc(sizeof(AVPacket));
    // （和c不一样的是，ffmpeg初始化参数的时候都有特定的方法去初始化）
    av_init_packet(avPacket);

    // 初始化AvFrame
    // This structure describes decoded (raw) audio or video data.
    // 这个结构描述了解码（原始的）音频或视频数据。
    // 这里需要将AvPacket(帧)里面的数据解析到AvFrame中去。（这里会在avcodec_decode_video2方法讲AvPacket的方法解析进去）
    AVFrame *srcFrame = av_frame_alloc();

    // 初始化 目标 Frame
    AVFrame *dstFrame = av_frame_alloc();
    // dstFrame分配内存
    u_int8_t *out_buffer = (u_int8_t *) av_malloc(
            avpicture_get_size(AV_PIX_FMT_RGBA, avCodecContext->width, avCodecContext->height));
    // 基于指定的图像参数设置图像字段
    // 以及所提供的图像数据缓冲区。
    avpicture_fill((AVPicture *) (dstFrame), out_buffer, AV_PIX_FMT_RGBA, avCodecContext->width,
                   avCodecContext->height);

    // 提供对本机窗口的访问
    ANativeWindow *nativeWindow = ANativeWindow_fromSurface(env, surface);
    // 视频缓冲区
    ANativeWindow_Buffer outBuffer;
    // ANativeWindow

    int got_frame;
    // 将视频数据读取到avPacket（avPacket代表一帧，这里的while循环也是一帧一帧读）
    while (av_read_frame(avFormatContext, avPacket) >= 0) {

        //  Decode the video frame of size avpkt->size from avpkt->data into picture.
        //  Some decoders may support multiple frames in a single AVPacket,
        //  such decoders would then just decode the first frame.
        // 将avPacket中的数据解析到srcFrame中去
        avcodec_decode_video2(avCodecContext, srcFrame, &got_frame, avPacket);
        // got_frame got_picture_ptr Zero if no frame could be decompressed, otherwise, it is nonzero.
        // 如果没有帧可以解压，则为0，否则为非0。
        if (got_frame) {
            // 设置缓存区
            ANativeWindow_setBuffersGeometry(nativeWindow, avCodecContext->width,
                                             avCodecContext->height, WINDOW_FORMAT_RGBA_8888);
            ANativeWindow_lock(nativeWindow, &outBuffer, NULL);

            // 将h264的格式转化成rgb
            // 从srcFrame中的数据（h264）解析成rgb存放到dstFrame中去
            sws_scale(swsContext, (const uint8_t *const *) srcFrame->data, srcFrame->linesize, 0,
                      srcFrame->height, dstFrame->data,
                      dstFrame->linesize
            );

            // 一帧的具体字节大小
            uint8_t *dst = static_cast<uint8_t *>(outBuffer.bits);
            // 每一个像素的字节  ARGB 一共是四个字节
            int dstStride = outBuffer.stride * 4;

            // 像素数据的首地址
            uint8_t *src = dstFrame->data[0];
            int srcStride = dstFrame->linesize[0];

            // 将 dstFrame的数据 一行行复制到屏幕上去
            for (int i = 0; i < avCodecContext->height; ++i) {
                memcpy(dst + i * dstStride, src + i * srcStride, srcStride);
            }
            ANativeWindow_unlockAndPost(nativeWindow);

            // 16微秒之后解析下一帧，因为人的眼睛无法观测到这个变化
            usleep(1000 * 16);
        }
        // 绘制完成之后，回收一帧资源Packet
        av_free_packet(avPacket);
    }
    // 回收窗体
    ANativeWindow_release(nativeWindow);

    // 回收Frame
    av_frame_free(&srcFrame);
    av_frame_free(&dstFrame);
    // 关闭解码器资
    avcodec_close(avCodecContext);
    avformat_free_context(avFormatContext);

    env->ReleaseStringUTFChars(path_, path);
}
```
读取视频时记得加上权限！

最后竟然能跑起来，真是不可思议。
![最终播放效果](https://upload-images.jianshu.io/upload_images/1930161-418c0ff53973f3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我其他的FFmpeg的博客：
* [FFmpeg—Linux编译FFmpeg动态库（一）](https://www.jianshu.com/p/e4b862e6fd7e)   
* [FFmpeg—项目导入FFmpeg动态库（二）](https://www.jianshu.com/p/2432cd236ffe)

代码地址:https://github.com/AxeChen/FFmpegStudy
 














