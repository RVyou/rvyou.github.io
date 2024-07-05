---
title: ffmpeg 6.0 中 AVFrame 结构体
author: rvyou
date: 2023-08-22 20:15:30
categories: [audio_and_video,ffmpeg,AVFrame]
tags: [audio_and_video,ffmpeg,AVFrame]
---
![Desktop View](assets/img/ffmpeg.png)
AVFrame 是 ffmpeg 存储原始数据。

## AVFrame

```c
typedef struct AVFrame {
#define AV_NUM_DATA_POINTERS 8
    /**
     * 指向图片/通道平面的指针。
     * 这可能与第一个分配的字节不同。对于视频，
     * 它甚至可以指向图像数据的末尾。
     * 
     * data 和 extended_data 中的所有指针都必须指向 buf 或 extended_buf 中的某个 AVBufferRef。
     * 
     * 一些解码器访问 0,0 - width,height 之外的区域，请
     * 参阅 avcodec_align_dimensions2()。一些过滤器和 swscale 可以读取
     * 超出平面 16 个字节，如果要使用这些过滤器，
     * 则必须分配 16 个额外的字节。
     * 
     * 注意：格式不需要的指针必须设置为 NULL。
     * 
     * @注意 在视频情况下，data[] 指针可以指向图像数据的末尾，以便反转行顺序，当与 linesize[] 数组中的负值结合使用时。
     */
    uint8_t *data[AV_NUM_DATA_POINTERS];

    /**
     * 对于视频，一个正值或负值，通常表示每个图片行的字节大小，但它也可以是：
     * - 用于垂直翻转的行的负字节大小
     *   （其中 data[n] 指向数据的末尾
     * - 正值或负值乘以字节大小，如用于访问
     *   帧的偶数和奇数场（可能翻转）
     * 
     * 对于音频，只能设置 linesize[0]。对于平面音频，每个通道
     * 平面必须具有相同的大小。
     * 
     * 对于视频，linesize 应该是 CPU 对齐首选项的倍数，对于现代桌面 CPU，这通常是 16 或 32。
     * 一些代码需要这种对齐，其他代码在没有的情况下可能会变慢
     * 正确的对齐，对于其他一些代码，则没有区别。
     * 
     * @注意 linesize 可能大于可用数据的尺寸 - 由于性能原因，可能存在额外的填充。
     * 
     * @注意 在视频情况下，行大小值可以为负，以实现
     * 图像行的垂直反转迭代。
     */
    int linesize[AV_NUM_DATA_POINTERS];

    /**
     * 指向数据平面/通道的指针。
     * 
     * 对于视频，这应该简单地指向 data[]。
     * 
     * 对于平面音频，每个通道都有一个单独的数据指针，并且
     * linesize[0] 包含每个通道缓冲区的大小。
     * 对于打包音频，只有一个数据指针，并且 linesize[0]
     * 包含所有通道的缓冲区的总大小。
     * 
     * 注意：data 和 extended_data 应该始终在有效帧中设置，
     * 但是对于具有比 data 中可以容纳的通道数量更多的平面音频，
     * 必须使用 extended_data 来访问所有通道。
     */
    uint8_t **extended_data;

    /**
     * @name 视频尺寸
     * 仅限视频帧。视频帧的编码尺寸（以像素为单位），
     * 即包含一些定义明确值的矩形的大小。
     * 
     * @注意 帧中用于显示/呈现的部分进一步
     * 受 @ref 裁剪 "裁剪矩形" 的限制。
     * @{
     */
    int width, height;
    /**
     * @}
     */

    /**
     * 此帧所描述的音频样本数量（每个通道）。
     */
    int nb_samples;

    /**
     * 帧的格式，-1 表示未知或未设置
     * 值对应于视频帧的 enum AVPixelFormat，
     * 音频的 enum AVSampleFormat）
     */
    int format;

    /**
     * 1 -> 关键帧，0-> 非关键帧
     */
    int key_frame;

    /**
     * 帧的图片类型。
     */
    enum AVPictureType pict_type;

    /**
     * 视频帧的样本纵横比，如果未知/未指定则为 0/1。
     */
    AVRational sample_aspect_ratio;

    /**
     * 以 time_base 单位表示的呈现时间戳（帧应显示给用户的时机）。
     */
    int64_t pts;

    /**
     * 从触发返回此帧的 AVPacket 中复制的 DTS。（如果未使用帧线程）
     * 这也是从
     * 仅 AVPacket.dts 值计算出的此 AVFrame 的呈现时间，没有 pts 值。
     */
    int64_t pkt_dts;

    /**
     * 此帧中时间戳的时间基准。
     * 未来，此字段可能在解码器或
     * 过滤器输出的帧上设置，但其值默认情况下将被忽略，不会输入编码器
     * 或过滤器。
     */
    AVRational time_base;

#if FF_API_FRAME_PICTURE_NUMBER
    /**
     * 比特流顺序中的图片编号
     */
    attribute_deprecated
    int coded_picture_number;
    /**
     * 显示顺序中的图片编号
     */
    attribute_deprecated
    int display_picture_number;
#endif

    /**
     * 质量（介于 1（好）和 FF_LAMBDA_MAX（差）之间）
     */
    int quality;

    /**
     * 用于用户的某些私有数据
     */
    void *opaque;

    /**
     * 解码时，这表示图片必须延迟多少。
     * extra_delay = repeat_pict / (2*fps)
     */
    int repeat_pict;

    /**
     * 图片的内容是隔行扫描的。
     */
    int interlaced_frame;

    /**
     * 如果内容是隔行扫描的，则顶部场先显示。
     */
    int top_field_first;

    /**
     * 告诉用户应用程序调色板已从前一帧更改。
     */
    int palette_has_changed;

#if FF_API_REORDERED_OPAQUE
    /**
     * 重新排序的不透明 64 位（通常是整数或双精度浮点数
     * PTS，但可以是任何值）。
     * 用户将 AVCodecContext.reordered_opaque 设置为代表当时的输入，
     * 解码器根据需要重新排序值，并将 AVFrame.reordered_opaque
     * 设置为用户通过 AVCodecContext.reordered_opaque 提供的值之一
     * 
     * @deprecated 使用 AV_CODEC_FLAG_COPY_OPAQUE 代替
     */
    attribute_deprecated
    int64_t reordered_opaque;
#endif

    /**
     * 音频数据的采样率。
     */
    int sample_rate;

#if FF_API_OLD_CHANNEL_LAYOUT
    /**
     * 音频数据的通道布局。
     * @deprecated 使用 ch_layout 代替
     */
    attribute_deprecated
    uint64_t channel_layout;
#endif

    /**
     * 支持此帧数据的 AVBuffer 引用。data 和 extended_data 中的所有指针都必须指向 buf 或 extended_buf 中的某个缓冲区。此数组必须连续填充 - 如果 buf[i] 不为 NULL，则对于所有 j < i，buf[j] 也必须不为 NULL。
     * 
     * 每个数据平面最多可能只有一个 AVBuffer，因此对于视频，此数组始终包含所有引用。对于具有超过 AV_NUM_DATA_POINTERS 个通道的平面音频，缓冲区数量可能超过此数组中可以容纳的数量。然后，额外的 AVBufferRef 指针将存储在 extended_buf 数组中。
     */
    AVBufferRef *buf[AV_NUM_DATA_POINTERS];

    /**
     * 对于需要超过 AV_NUM_DATA_POINTERS 个 AVBufferRef 指针的平面音频，此数组将保存所有无法放入 AVFrame.buf 的引用。
     * 
     * 请注意，这与 AVFrame.extended_data 不同，AVFrame.extended_data 始终包含所有指针。此数组仅包含无法放入 AVFrame.buf 的额外指针。
     * 
     * 此数组始终由构造帧的任何人使用 av_malloc() 分配。它在 av_frame_unref() 中释放。
     */
    AVBufferRef **extended_buf;
    /**
     * extended_buf 中的元素数量。
     */
    int        nb_extended_buf;

    AVFrameSideData **side_data;
    int            nb_side_data;

/**
 * @defgroup lavu_frame_flags AV_FRAME_FLAGS
 * @ingroup lavu_frame
 * 描述附加帧属性的标志。
 * 
 * @{
 */

/**
 * 帧数据可能已损坏，例如，由于解码错误。
 */
#define AV_FRAME_FLAG_CORRUPT       (1 << 0)
/**
 * 用于标记需要解码但不能输出的帧的标志。
 */
#define AV_FRAME_FLAG_DISCARD   (1 << 2)
/**
 * @}
 */

    /**
     * 帧标志，@ref lavu_frame_flags 的组合
     */
    int flags;

    /**
     * MPEG 与 JPEG YUV 范围。
     * - 编码：由用户设置
     * - 解码：由 libavcodec 设置
     */
    enum AVColorRange color_range;

    enum AVColorPrimaries color_primaries;

    enum AVColorTransferCharacteristic color_trc;

    /**
     * YUV 颜色空间类型。
     * - 编码：由用户设置
     * - 解码：由 libavcodec 设置
     */
    enum AVColorSpace colorspace;

    enum AVChromaLocation chroma_location;

    /**
     * 使用各种启发式方法估算的帧时间戳，以流时间基准为单位
     * - 编码：未使用
     * - 解码：由 libavcodec 设置，由用户读取。
     */
    int64_t best_effort_timestamp;

    /**
     * 从最后输入解码器的 AVPacket 中重新排序的 pos
     * - 编码：未使用
     * - 解码：由用户读取。
     */
    int64_t pkt_pos;

#if FF_API_PKT_DURATION
    /**
     * 对应数据包的持续时间，以 AVStream->time_base 单位表示，如果未知则为 0。
     * - 编码：未使用
     * - 解码：由用户读取。
     * 
     * @deprecated 使用 duration 代替
     */
    attribute_deprecated
    int64_t pkt_duration;
#endif

    /**
     * 元数据。
     * - 编码：由用户设置。
     * - 解码：由 libavcodec 设置。
     */
    AVDictionary *metadata;

    /**
     * 帧的解码错误标志，如果解码器生成了帧，则设置为 FF_DECODE_ERROR_xxx 标志的组合，但解码期间存在错误。
     * - 编码：未使用
     * - 解码：由 libavcodec 设置，由用户读取。
     */
    int decode_error_flags;
#define FF_DECODE_ERROR_INVALID_BITSTREAM   1
#define FF_DECODE_ERROR_MISSING_REFERENCE   2
#define FF_DECODE_ERROR_CONCEALMENT_ACTIVE  4
#define FF_DECODE_ERROR_DECODE_SLICES       8

#if FF_API_OLD_CHANNEL_LAYOUT
    /**
     * 音频通道数量，仅用于音频。
     * - 编码：未使用
     * - 解码：由用户读取。
     * @deprecated 使用 ch_layout 代替
     */
    attribute_deprecated
    int channels;
#endif

    /**
     * 包含压缩的对应数据包的大小
     * 帧。
     * 如果未知，则设置为负值。
     * - 编码：未使用
     * - 解码：由 libavcodec 设置，由用户读取。
     */
    int pkt_size;

    /**
     * 对于 hwaccel 格式帧，这应该是描述该帧的 AVHWFramesContext 的引用。
     */
    AVBufferRef *hw_frames_ctx;

    /**
     * 供 API 用户自由使用的 AVBufferRef。FFmpeg 永远不会检查
     * 缓冲区引用的内容。FFmpeg 在
     * 帧未被引用时对它调用 av_buffer_unref()。av_frame_copy_props() 调用使用 av_buffer_ref() 为目标帧的 opaque_ref 字段创建一个新的引用。
     * 
     * 这与 opaque 字段无关，尽管它具有类似的用途。
     */
    AVBufferRef *opaque_ref;

    /**
     * @anchor 裁剪
     * @name 裁剪
     * 仅限视频帧。从帧的顶部/底部/左侧/右侧边框丢弃的像素数量，以获得帧的用于呈现的子矩形。
     * @{
     */
    size_t crop_top;
    size_t crop_bottom;
    size_t crop_left;
    size_t crop_right;
    /**
     * @}
     */

    /**
     * 供单个 libav* 库内部使用的 AVBufferRef。
     * 不得用于在库之间传输数据。
     * 帧的所有权离开相应库时，必须为 NULL。
     * 
     * FFmpeg 库之外的代码永远不应该检查或更改缓冲区引用的内容。
     * 
     * FFmpeg 在帧未被引用时对它调用 av_buffer_unref()。
     * av_frame_copy_props() 调用使用 av_buffer_ref() 为目标帧的 private_ref 字段创建一个新的引用。
     */
    AVBufferRef *private_ref;

    /**
     * 音频数据的通道布局。
     */
    AVChannelLayout ch_layout;

    /**
     * 帧的持续时间，以与 pts 相同的单位表示。如果未知则为 0。
     */
    int64_t duration;
} AVFrame;
```


