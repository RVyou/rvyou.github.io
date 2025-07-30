---
title: FFmpeg 6.0 中 AVCodec 结构体
author: rvyou
date: 2023-08-22 20:25:30
categories: [audio_and_video,FFmpeg,AVCodec]
tags: [audio_and_video,FFmpeg,AVCodec]
---
![Desktop View](assets/img/ffmpeg.png)
AVCodec 是 FFmpeg 存储编解码器信息。

## AVCodec

```c
typedef struct AVCodec {
    /**
     * 编解码器实现的名称。
     * 该名称在编码器之间和解码器之间是全局唯一的（但编码器和解码器可以共享相同的名称）。
     * 这是从用户角度查找编解码器的主要方法。
     */
    const char *name;
    /**
     * 编解码器的描述性名称，旨在比名称更易于人类阅读。
     * 您应该使用 NULL_IF_CONFIG_SMALL() 宏来定义它。
     */
    const char *long_name;
    enum AVMediaType type;
    enum AVCodecID id;
    /**
     * 编解码器功能。
     * 参阅 AV_CODEC_CAP_*
     */
    int capabilities;
    uint8_t max_lowres;                     ///< 解码器支持的 lowres 的最大值
    const AVRational *supported_framerates; ///< 支持的帧速率数组，如果任意帧速率都支持则为 NULL，数组以 {0,0} 结尾
    const enum AVPixelFormat *pix_fmts;     ///< 支持的像素格式数组，如果未知则为 NULL，数组以 -1 结尾
    const int *supported_samplerates;       ///< 支持的音频采样率数组，如果未知则为 NULL，数组以 0 结尾
    const enum AVSampleFormat *sample_fmts; ///< 支持的样本格式数组，如果未知则为 NULL，数组以 -1 结尾
#if FF_API_OLD_CHANNEL_LAYOUT
    /**
     * @deprecated 使用 ch_layouts 代替
     */
    attribute_deprecated
    const uint64_t *channel_layouts;         ///< 支持的通道布局数组，如果未知则为 NULL。数组以 0 结尾
#endif
    const AVClass *priv_class;              ///< 私有上下文的 AVClass
    const AVProfile *profiles;              ///< 识别的配置文件数组，如果未知则为 NULL，数组以 {FF_PROFILE_UNKNOWN} 结尾

    /**
     * 编解码器实现的组名。
     * 这是支持此编解码器的包装器的简短符号名称。
     * 包装器使用某种外部实现来实现编解码器，例如
     * 外部库，或者操作系统或
     * 硬件提供的编解码器实现。
     * 如果此字段为 NULL，则这是一个内置的、libavcodec 本地编解码器。
     * 如果不为 NULL，则在大多数情况下，这将是 AVCodec.name 中的后缀
     * （通常 AVCodec.name 将采用 "<编解码器名称>_<包装器名称>" 的形式）。
     */
    const char *wrapper_name;

    /**
     * 支持的通道布局数组，以一个为零的布局结尾。
     */
    const AVChannelLayout *ch_layouts;
} AVCodec;
```



