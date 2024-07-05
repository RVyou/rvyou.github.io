---
title: ffmpeg 6.0 中 AVCodecContext 结构体
author: rvyou
date: 2023-08-22 19:52:30
categories: [audio_and_video,ffmpeg,AVCodecContext]
tags: [audio_and_video,ffmpeg,AVCodecContext]
---
![Desktop View](assets/img/ffmpeg.png)
AVCodecContext 是 ffmpeg 编解码器。

## AVCodecContext

```c
typedef struct AVCodecContext {
    const AVClass *av_class;//AVClass用于打印log和AVOption的引用，由avcodec_alloc_context3() 函数创建
    int log_level_offset;
    enum AVMediaType codec_type; /* see AVMEDIA_TYPE_xxx */
    const struct AVCodec *codec;
    enum AVCodecID codec_id; /* see AV_CODEC_ID_xxx */
    //编码： 由用户设置，否则将使用基于 codec_id 的默认编码。- 解码： 由用户设置，libavcodec 将在启动时将其转换为大写
    unsigned int codec_tag;
    void *priv_data;
    struct AVCodecInternal *internal; //用于内部数据的专用上下文。与 priv_data 不同，它不针对特定的编解码器。它用于一般libavcodec 函数。

    void *opaque; //用户自定义数据，可用于承载应用程序的特定内容。


    /**
     * 平均码率
     * - 编码：由用户设置；对于恒定量化器编码不使用。
     * - 解码：由用户设置，如果流中存在此信息，则可能被 libavcodec 覆盖。
     */
    int64_t bit_rate;

    /**
     * 比特流允许偏离参考的位数。
     * 参考可以是 CBR（对于 CBR 第 1 遍）或 VBR（对于第 2 遍）。
     * - 编码：由用户设置；对于恒定量化器编码不使用。
     * - 解码：不使用。
     */
    int bit_rate_tolerance;

    /**
     * 无法按帧更改的编解码器的全局质量。
     * 这应该与 MPEG-1/2/4 qscale 成比例。
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int global_quality;

    /**
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int compression_level;
#define FF_COMPRESSION_DEFAULT -1

    /**
     * AV_CODEC_FLAG_*。
     * - 编码：由用户设置。
     * - 解码：由用户设置。
     */
    int flags;

    /**
     * AV_CODEC_FLAG2_*
     * - 编码：由用户设置。
     * - 解码：由用户设置。
     */
    int flags2;

    /**
     * 一些编解码器需要/可以使用额外的信息，如霍夫曼表。
     * MJPEG：霍夫曼表
     * rv10：附加标志
     * MPEG-4：全局头（它们可以位于比特流中或这里）
     * 分配的内存应比 extradata_size 大 AV_INPUT_BUFFER_PADDING_SIZE 字节，以避免在使用比特流读取器读取时出现问题。
     * extradata 的字节内容不能依赖于架构或 CPU 字节序。
     * 必须使用 av_malloc() 函数族进行分配。
     * - 编码：由 libavcodec 设置/分配/释放。
     * - 解码：由用户设置/分配/释放。
     */
    uint8_t *extradata;
    int extradata_size;

    /**
     * 这是以秒为单位的基本时间单位，帧时间戳以它为单位表示。对于固定帧率的内容，
     * timebase 应该为 1/帧率，时间戳增量应该始终为 1。
     * 这通常（但并非总是）是视频帧率或场率的倒数。如果帧率不是
     * 恒定，则 1/time_base 不是平均帧率。
     *
     * 与容器类似，基本流也可以存储时间戳，1/time_base
     * 是指定这些时间戳的单位。
     * 作为这种编解码器时间基的示例，请参见 ISO/IEC 14496-2:2001(E)
     * vop_time_increment_resolution 和 fixed_vop_rate
     * (fixed_vop_rate == 0 意味着它与帧率不同)
     *
     * - 编码：必须由用户设置。
     * - 解码：不使用。
     */
    AVRational time_base;

    /**
     * 对于某些编解码器，时间基更接近场率而不是帧率。
     * 最值得注意的是，H.264 和 MPEG-2 指定 time_base 为帧持续时间的一半，
     * 如果没有使用隔行扫描 ...
     *
     * 设置为每帧的时间基刻度。默认值为 1，例如，H.264/MPEG-2 将其设置为 2。
     */
    int ticks_per_frame;

    /**
     * 编解码器延迟。
     *
     * 编码：从编码器输入到
     *           解码器输出的帧延迟数。（我们假设解码器与规范匹配）
     * 解码：除了标准解码器之外的帧延迟数
     *           如规范中所述。
     *
     * 视频：
     *   解码输出相对于编码输入的帧延迟数。
     *
     * 音频：
     *   对于编码，此字段不使用（请参见 initial_padding）。
     *
     *   对于解码，这是解码器在解码器输出有效之前需要输出的样本数。当进行查找时，您应该
     *   在您想要的查找点之前开始解码这么多个样本。
     *
     * - 编码：由 libavcodec 设置。
     * - 解码：由 libavcodec 设置。
     */
    int delay;


    /* 仅限视频 */
    /**
     * 图片宽度/高度。
     *
     * @注意 这些字段可能与 avcodec_receive_frame() 输出的最后一个
     * AVFrame 的值不匹配，因为帧
     * 重新排序。
     *
     * - 编码：必须由用户设置。
     * - 解码：可以在打开解码器之前由用户设置，如果已知，例如
     *             来自容器。一些解码器将要求调用者设置尺寸。在解码期间，解码器可能会
     *             根据需要覆盖这些值，同时解析数据。
     */
    int width, height;

    /**
     * 比特流宽度/高度，可能与 width/height 不同，例如当
     * 解码帧在输出之前被裁剪或启用了低分辨率时。
     *
     * @注意 这些字段可能与 avcodec_receive_frame() 输出的最后一个
     * AVFrame 的值不匹配，因为帧
     * 重新排序。
     *
     * - 编码：不使用
     * - 解码：可以在打开解码器之前由用户设置，如果已知，例如
     *             来自容器。在解码期间，解码器可能会
     *             根据需要覆盖这些值，同时解析数据。
     */
    int coded_width, coded_height;

    /**
     * 一组图片中的图片数量，或 0 表示仅限帧内。
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int gop_size;

    /**
     * 像素格式，请参见 AV_PIX_FMT_xxx。
     * 如果从头文件中已知，则可以由解复用器设置。
     * 如果解码器知道得更多，则可以由解码器覆盖。
     *
     * @注意 此字段可能与 avcodec_receive_frame() 输出的最后一个
     * AVFrame 的值不匹配，因为帧
     * 重新排序。
     *
     * - 编码：由用户设置。
     * - 解码：如果已知，则由用户设置，由 libavcodec 在
     *             解析数据时覆盖。
     */
    enum AVPixelFormat pix_fmt;

    /**
     * 如果非 NULL，则 'draw_horiz_band' 会由 libavcodec
     * 解码器调用以绘制水平带。它提高了缓存使用率。不是
     * 所有编解码器都能做到。您必须检查编解码器功能
     * 事先。
     * 当使用多线程时，它可能从多个线程
     * 同时调用；线程可能绘制同一 AVFrame 的不同部分，
     * 或多个 AVFrame，并且不能保证切片将按顺序绘制
     * 。
     * 该函数也被硬件加速 API 使用。
     * 它在帧解码期间至少被调用一次以传递
     * 硬件渲染所需的数据。
     * 在该模式下，AVFrame 指向
     * 而不是像素数据，而是指向特定于加速 API 的结构。应用程序
     * 读取结构，并可以更改一些字段以指示进度
     * 或标记状态。
     * - 编码：不使用
     * - 解码：由用户设置。
     * @param height 切片的高度
     * @param y 切片的 y 位置
     * @param type 1->顶部场，2->底部场，3->帧
     * @param offset 切片应从中读取的 AVFrame.data 的偏移量
     */
    void (*draw_horiz_band)(struct AVCodecContext *s,
                            const AVFrame *src, int offset[AV_NUM_DATA_POINTERS],
                            int y, int type, int height);

    /**
     * 回调以协商像素格式。仅限解码，可以在
     * avcodec_open2() 之前由调用者设置。
     *
     * 由一些解码器调用以选择将用于
     * 输出帧的像素格式。这主要用于设置硬件加速，
     * 然后提供的格式列表包含相应的 hwaccel 像素
     * 格式以及“软件”格式。软件像素格式也可以
     * 从 \ref sw_pix_fmt 中检索。
     *
     * 当编码帧属性（如
     * 分辨率、像素格式等）更改并且对于这些新属性支持多种输出格式时，将调用此回调。如果选择了硬件像素格式
     * 并且对其进行初始化失败，则可能立即再次调用回调。
     *
     * 如果解码器是
     * 多线程的，则此回调可能从不同的线程调用，但不能从多个线程同时调用。不需要是
     * 可重入的。
     *
     * @param fmt 可以用于当前的格式列表
     *            配置，以 AV_PIX_FMT_NONE 结尾。
     * @warning 如果回调返回的值不是
     *          fmt 中的格式或 AV_PIX_FMT_NONE 之一，则行为未定义。
     * @return 所选格式或 AV_PIX_FMT_NONE
     */
    enum AVPixelFormat (*get_format)(struct AVCodecContext *s, const enum AVPixelFormat *fmt);

    /**
     * 非 B 帧之间 B 帧的最大数量
     * 注意：输出将相对于输入延迟 max_b_frames+1。
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int max_b_frames;

    /**
     * IP 帧和 B 帧之间的 qscale 因子
     * 如果 > 0，则将使用最后一个 P 帧量化器（q= lastp_q*factor+offset）。
     * 如果 < 0，则将进行正常的速率控制（q= -normal_q*factor+offset）。
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float b_quant_factor;

    /**
     * IP 帧和 B 帧之间的 qscale 偏移量
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float b_quant_offset;

    /**
     * 解码器中帧重新排序缓冲区的大小。
     * 对于 MPEG-2，它为 1 IPB 或 0 低延迟 IP。
     * - 编码：由 libavcodec 设置。
     * - 解码：由 libavcodec 设置。
     */
    int has_b_frames;

    /**
     * P 帧和 I 帧之间的 qscale 因子
     * 如果 > 0，则将使用最后一个 P 帧量化器（q = lastp_q * factor + offset）。
     * 如果 < 0，则将进行正常的速率控制（q= -normal_q*factor+offset）。
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float i_quant_factor;

    /**
     * P 帧和 I 帧之间的 qscale 偏移量
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float i_quant_offset;

    /**
     * 亮度掩蔽（0-> 禁用）
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float lumi_masking;

    /**
     * 临时复杂度掩蔽（0-> 禁用）
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float temporal_cplx_masking;

    /**
     * 空间复杂度掩蔽（0-> 禁用）
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float spatial_cplx_masking;

    /**
     * p 块掩蔽（0-> 禁用）
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float p_masking;

    /**
     * 黑度掩蔽（0-> 禁用）
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float dark_masking;

    /**
     * 切片数量
     * - 编码：由 libavcodec 设置。
     * - 解码：由用户设置（或 0）。
     */
    int slice_count;

    /**
     * 帧中切片的偏移量（以字节为单位）
     * - 编码：由 libavcodec 设置/分配。
     * - 解码：由用户设置/分配（或 NULL）。
     */
    int *slice_offset;

    /**
     * 样本纵横比（如果未知则为 0）
     * 也就是说，像素的宽度除以像素的高度。
     * 对于某些视频标准，分子和分母必须互质且小于 256。
     * - 编码：由用户设置。
     * - 解码：由 libavcodec 设置。
     */
    AVRational sample_aspect_ratio;

    /**
     * 运动估计比较函数
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int me_cmp;
    /**
     * 子像素运动估计比较函数
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int me_sub_cmp;
    /**
     * 宏块比较函数（目前不支持）
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int mb_cmp;
    /**
     * 隔行 DCT 比较函数
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int ildct_cmp;

    /**
     * ME 菱形大小和形状
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int dia_size;

    /**
     * 以前的 MV 预测器的数量（2a+1 x 2a+1 正方形）
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int last_predictor_count;

    /**
     * 运动估计预处理比较函数
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int me_pre_cmp;

    /**
     * ME 预处理菱形大小和形状
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int pre_dia_size;

    /**
     * 子像素 ME 质量
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int me_subpel_quality;

    /**
     * 子像素单位中最大运动估计搜索范围
     * 如果为 0，则没有限制。
     *
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int me_range;

    /**
     * 切片标志
     * - 编码：不使用
     * - 解码：由用户设置。
     */
    int slice_flags;

    /**
     * 宏块决策模式
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int mb_decision;

    /**
     * 自定义帧内量化矩阵
     * 必须使用 av_malloc() 函数族进行分配，并在
     * avcodec_free_context() 中释放。
     * - 编码：由用户设置/分配，由 libavcodec 释放。可以为 NULL。
     * - 解码：由 libavcodec 设置/分配/释放。
     */
    uint16_t *intra_matrix;

    /**
     * 自定义帧间量化矩阵
     * 必须使用 av_malloc() 函数族进行分配，并在
     * avcodec_free_context() 中释放。
     * - 编码：由用户设置/分配，由 libavcodec 释放。可以为 NULL。
     * - 解码：由 libavcodec 设置/分配/释放。
     */
    uint16_t *inter_matrix;

    /**
     * 帧内 DC 系数的精度 - 8
     * - 编码：由用户设置。
     * - 解码：由 libavcodec 设置。
     */
    int intra_dc_precision;

    /**
     * 顶部跳过的宏块行数。
     * - 编码：不使用
     * - 解码：由用户设置。
     */
    int skip_top;

    /**
     * 底部跳过的宏块行数。
     * - 编码：不使用
     * - 解码：由用户设置。
     */
    int skip_bottom;

    /**
     * 最小 MB 拉格朗日乘数
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int mb_lmin;

    /**
     * 最大 MB 拉格朗日乘数
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int mb_lmax;

    /**
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int bidir_refine;

    /**
     * 最小 GOP 大小
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int keyint_min;

    /**
     * 参考帧的数量
     * - 编码：由用户设置。
     * - 解码：由 lavc 设置。
     */
    int refs;

    /**
     * 注意：值取决于用于全像素 ME 的比较函数。
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int mv0_threshold;

    /**
     * 源原色的色度坐标。
     * - 编码：由用户设置
     * - 解码：由 libavcodec 设置
     */
    enum AVColorPrimaries color_primaries;

    /**
     * 色彩转换特性。
     * - 编码：由用户设置
     * - 解码：由 libavcodec 设置
     */
    enum AVColorTransferCharacteristic color_trc;

    /**
     * YUV 色彩空间类型。
     * - 编码：由用户设置
     * - 解码：由 libavcodec 设置
     */
    enum AVColorSpace colorspace;

    /**
     * MPEG 与 JPEG YUV 范围。
     * - 编码：由用户设置
     * - 解码：由 libavcodec 设置
     */
    enum AVColorRange color_range;

    /**
     * 这定义了色度样本的位置。
     * - 编码：由用户设置
     * - 解码：由 libavcodec 设置
     */
    enum AVChromaLocation chroma_sample_location;

    /**
     * 切片数量。
     * 指示图片细分的数量。用于并行
     * 解码。
     * - 编码：由用户设置
     * - 解码：不使用
     */
    int slices;

    /** 场序
     * - 编码：由 libavcodec 设置
     * - 解码：由用户设置。
     */
    enum AVFieldOrder field_order;

    /* 仅限音频 */
    int sample_rate; ///< 每秒样本数


    /**
     * 音频通道数量
     * @已弃用 使用 ch_layout.nb_channels
     */
    attribute_deprecated
    int channels;


    /**
     * 音频样本格式
     * - 编码：由用户设置。
     * - 解码：由 libavcodec 设置。
     */
    enum AVSampleFormat sample_fmt;  ///< 样本格式

    /* 以下数据不应初始化。 */
    /**
     * 音频帧中每个通道的样本数。
     *
     * - 编码：在 avcodec_open2() 中由 libavcodec 设置。除最后一个帧之外的每个提交的帧
     *   必须包含每个通道正好 frame_size 个样本。
     *   如果编解码器已设置 AV_CODEC_CAP_VARIABLE_FRAME_SIZE，则可能为 0，此时帧大小不受限制。
     * - 解码：可能由一些解码器设置以指示恒定帧大小。
     */
    int frame_size;


    /**
     * 帧计数器，由 libavcodec 设置。
     *
     * - 解码：到目前为止从解码器返回的帧总数。
     * - 编码：到目前为止传递给编码器的帧总数。
     *
     *   @注意 如果编码/解码导致
     *   错误，则计数器不会递增。
     *   @已弃用 使用 frame_num 代替。
     */
    attribute_deprecated
    int frame_number;


    /**
     * 如果已知且恒定，则每个数据包的字节数，否则为 0
     * 由某些基于 WAV 的音频编解码器使用。
     */
    int block_align;

    /**
     * 音频截止带宽（0 表示“自动”）
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int cutoff;


    /**
     * 音频通道布局。
     * - 编码：由用户设置。
     * - 解码：由用户设置，可能被 libavcodec 覆盖。
     * @已弃用 使用 ch_layout
     */
    attribute_deprecated
    uint64_t channel_layout;

    /**
     * 如果可以，请求解码器使用此通道布局（0 表示默认值）
     * - 编码：不使用
     * - 解码：由用户设置。
     * @已弃用 使用“downmix”编解码器专用选项
     */
    attribute_deprecated
    uint64_t request_channel_layout;


    /**
     * 音频流传递的服务类型。
     * - 编码：由用户设置。
     * - 解码：由 libavcodec 设置。
     */
    enum AVAudioServiceType audio_service_type;

    /**
     * 想要的样本格式
     * - 编码：不使用。
     * - 解码：由用户设置。
     * 如果可以，解码器将解码为此格式。
     */
    enum AVSampleFormat request_sample_fmt;

    /**
     * 此回调在每个帧的开头调用，以获取
     * 为其提供数据缓冲区。可能有一个连续的缓冲区用于所有数据，或者
     * 每个数据平面可能有一个缓冲区，或者介于两者之间。这意味着
     * 您可以在 buf[] 中设置任意数量的条目，只要您认为有必要。
     * 每个缓冲区必须使用 AVBuffer API 进行引用计数（请参见
     * 以下 buf[] 的描述）。
     *
     * 在调用此回调之前，将在帧中设置以下字段：
     * - format
     * - width, height（仅限视频）
     * - sample_rate, channel_layout, nb_samples（仅限音频）
     * 它们的值可能与
     * AVCodecContext 中的相应值不同。此回调必须使用帧值，而不是编解码器
     * 上下文值，来计算所需的缓冲区大小。
     *
     * 此回调必须在帧中填充以下字段：
     * - data[]
     * - linesize[]
     * - extended_data:
     *   * 如果数据是具有超过 8 个通道的平面音频，则此
     *     回调必须分配和填充 extended_data 以包含指向所有数据平面的所有指针。data[] 必须包含与它可以容纳的指针一样多的指针。
     *     extended_data 必须使用 av_malloc() 进行分配，并在
     *     av_frame_unref() 中释放。
     *   * 否则，extended_data 必须指向 data
     * - buf[] 必须包含一个或多个指向 AVBufferRef 结构的指针。帧的每个 data 和 extended_data 指针都必须包含在这些指针中。也就是说，每个分配的内存块对应一个 AVBufferRef，不一定每个 data[] 条目对应一个 AVBufferRef。请参见：av_buffer_create()、av_buffer_alloc()
     *   和 av_buffer_ref()。
     * - extended_buf 和 nb_extended_buf 必须由
     *   此回调使用 av_malloc() 进行分配，并用额外的缓冲区填充，如果缓冲区数量超过 buf[] 可以容纳的缓冲区数量。extended_buf 将在
     *   av_frame_unref() 中释放。
     *
     * 如果未设置 AV_CODEC_CAP_DR1，则 get_buffer2() 必须调用
     * avcodec_default_get_buffer2()，而不是提供由
     * 其他方法分配的缓冲区。
     *
     * 每个数据平面都必须与目标
     * CPU 所需的最大对齐方式对齐。
     *
     * @see avcodec_default_get_buffer2()
     *
     * 视频：
     *
     * 如果在 flags 中设置了 AV_GET_BUFFER_FLAG_REF，则 libavcodec 可能会在以后重用帧
     * （如果可写，则读入和/或写入）。
     *
     * 应使用 avcodec_align_dimensions2() 来查找所需的宽度和
     * 高度，因为它们通常需要向上取整到 16 的下一个倍数。
     *
     * 一些解码器不支持帧之间 linesizes 更改。
     *
     * 如果使用帧多线程，则此回调可能从一个
     * 不同的线程调用，但不能从多个线程同时调用。不需要是
     * 可重入的。
     *
     * @see avcodec_align_dimensions2()
     *
     * 音频：
     *
     * 解码器通过在调用 get_buffer2() 之前设置
     * AVFrame.nb_samples 来请求特定大小的缓冲区。然而，解码器可能
     * 只使用一部分缓冲区，通过在输出帧中将 AVFrame.nb_samples
     * 设置为较小的值。
     *
     * 为方便起见，libavutil 中的 av_samples_get_buffer_size() 和
     * av_samples_fill_arrays() 可以被自定义的 get_buffer2()
     * 函数使用，以查找所需的数据大小，并填充数据指针和
     * linesize。在 AVFrame.linesize 中，仅 linesize[0] 可以为音频设置，
     * 因为所有平面都必须具有相同的大小。
     *
     * @see av_samples_get_buffer_size(), av_samples_fill_arrays()
     *
     * - 编码：不使用
     * - 解码：由 libavcodec 设置，用户可以覆盖。
     */
    int (*get_buffer2)(struct AVCodecContext *s, AVFrame *frame, int flags);

    /* - 编码参数 */
    float qcompress;  ///< 简单场景和复杂场景之间的 qscale 变化量 (0.0-1.0)
    float qblur;      ///< qscale 在时间上的平滑量 (0.0-1.0)

    /**
     * 最小量化器
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int qmin;

    /**
     * 最大量化器
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int qmax;

    /**
     * 帧之间最大量化器差值
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int max_qdiff;

    /**
     * 解码器比特流缓冲区大小
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int rc_buffer_size;

    /**
     * 速率控制覆盖，请参见 RcOverride
     * - 编码：由用户分配/设置/释放。
     * - 解码：不使用。
     */
    int rc_override_count;
    RcOverride *rc_override;

    /**
     * 最大码率
     * - 编码：由用户设置。
     * - 解码：由用户设置，可能被 libavcodec 覆盖。
     */
    int64_t rc_max_rate;

    /**
     * 最小码率
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int64_t rc_min_rate;

    /**
     * 速率控制尝试使用，最多使用<value> 的可用量，而不会发生下溢。
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float rc_max_available_vbv_use;

    /**
     * 速率控制尝试使用，至少使用<value> 倍于防止 VBV 溢出所需的量。
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    float rc_min_vbv_overflow_use;

    /**
     * 开始解码之前应加载到 rc 缓冲区中的位数。
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int rc_initial_buffer_occupancy;

    /**
     * 格子 RD 量化
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int trellis;

    /**
     * 第 1 遍编码统计输出缓冲区
     * - 编码：由 libavcodec 设置。
     * - 解码：不使用。
     */
    char *stats_out;

    /**
     * 第 2 遍编码统计输入缓冲区
     * 来自第 1 遍的 stats_out 的串联内容应放置在这里。
     * - 编码：由用户分配/设置/释放。
     * - 解码：不使用。
     */
    char *stats_in;

    /**
     * 解决编码器中无法自动检测到的错误。
     * - 编码：由用户设置
     * - 解码：由用户设置
     */
    int workaround_bugs;

    /**
     * 严格遵循标准（MPEG-4，...）。
     * - 编码：由用户设置。
     * - 解码：由用户设置。
     * 将其设置为 STRICT 或更高意味着编码器和解码器通常会
     * 做一些愚蠢的事情，而将其设置为 unofficial 或更低
     * 意味着编码器可能会产生一些不受所有
     * 符合规范的解码器支持的输出。解码器不会区分 normal、
     * unofficial 和 experimental（也就是说，它们总是尝试在可以的情况下解码内容），除非它们被明确要求表现出愚蠢的行为
     * (=严格符合规范)
     * 这只能设置为 defs.h 中的 FF_COMPLIANCE_* 值之一。
     */
    int strict_std_compliance;

    /**
     * 错误隐藏标志
     * - 编码：不使用
     * - 解码：由用户设置。
     */
    int error_concealment;

    /**
     * 调试
     * - 编码：由用户设置。
     * - 解码：由用户设置。
     */
    int debug;

    /**
     * 错误识别；可能会将一些或多或少有效的部分误认为错误。
     * 这是一个由 defs.h 中定义的 AV_EF_* 值组成的位域。
     *
     * - 编码：由用户设置。
     * - 解码：由用户设置。
     */
    int err_recognition;

    /**
     * 不透明的 64 位数字（通常是 PTS），它将被重新排序并
     * 输出到 AVFrame.reordered_opaque 中
     * - 编码：由 libavcodec 设置为与最后一个返回的数据包对应的输入
     *             帧的 reordered_opaque。仅
     *             支持具有
     *             AV_CODEC_CAP_ENCODER_REORDERED_OPAQUE 功能的编码器。
     * - 解码：由用户设置。
     *
     * @已弃用 使用 AV_CODEC_FLAG_COPY_OPAQUE 代替。
     */
    attribute_deprecated
    int64_t reordered_opaque;
#endif

    /**
     * 正在使用的硬件加速器
     * - 编码：不使用。
     * - 解码：由 libavcodec 设置。
     */
    const struct AVHWAccel *hwaccel;

    /**
     * 传统硬件加速器上下文。
     *
     * 对于某些硬件加速方法，调用者可以使用此字段来
     * 向编解码器发送特定于 hwaccel 的数据。此指针指向的结构是特定于 hwaccel 的，并在相应的头文件中定义。请
     * 参照 FFmpeg 硬件加速文档以了解如何填写
     * 此项。
     *
     * 在大多数情况下，此字段是可选的 - 必要的信息也可以
     * 通过 @ref hw_frames_ctx 或 @ref
     * hw_device_ctx（参见 avcodec_get_hw_config()）提供给 libavcodec。但是，在某些情况下，它
     * 可能是发送一些（可选）信息的唯一方法。
     *
     * 结构及其内容归调用者所有。
     *
     * - 编码：可以在 avcodec_open2() 之前由调用者设置。必须保持
     *             有效，直到 avcodec_free_context() 为止。
     * - 解码：可以在 get_format() 回调中由调用者设置。
     *             必须保持有效，直到下一个 get_format() 调用为止，
     *             或 avcodec_free_context()（以先到者为准）。
     */
    void *hwaccel_context;

    /**
     * 错误
     * - 编码：如果 flags & AV_CODEC_FLAG_PSNR，则由 libavcodec 设置。
     * - 解码：不使用。
     */
    uint64_t error[AV_NUM_DATA_POINTERS];

    /**
     * DCT 算法，请参见下面的 FF_DCT_*
     * - 编码：由用户设置。
     * - 解码：不使用。
     */
    int dct_algo;

    /**
     * IDCT 算法，请参见下面的 FF_IDCT_*。
     * - 编码：由用户设置。
     * - 解码：由用户设置。
     */
    int idct_algo;

    /**
     * 来自解复用器的每个样本/像素的位数（对于 huffyuv 所需）。
     * - 编码：由 libavcodec 设置。
     * - 解码：由用户设置。
     */
    int bits_per_coded_sample;

    /**
     * 内部 libavcodec 像素/样本格式的每个样本/像素的位数。
     * - 编码：由用户设置。
     * - 解码：由 libavcodec 设置。
     */
    int bits_per_raw_sample;

    /**
     * 低分辨率解码，1-> 1/2 大小，2-> 1/4 大小
     * - 编码：不使用
     * - 解码：由用户设置。
     */
    int lowres;

    /**
     * 线程数量
     * 用于决定应将多少个独立的任务传递给 execute()
     * - 编码：由用户设置。
     * - 解码：由用户设置。
     */
    int thread_count;

    /**
     * 要使用的多线程方法。
     * 使用 FF_THREAD_FRAME 将使解码延迟增加每线程一帧，
     * 因此无法提供未来帧的客户端不应使用它。
     *
     * - 编码：由用户设置，否则使用默认值。
     * - 解码：由用户设置，否则使用默认值。
     */
    int thread_type;

    /**
     * 编解码器正在使用的多线程方法。
     * - 编码：由 libavcodec 设置。
     * - 解码：由 libavcodec 设置。
     */
    int active_thread_type;

    /**
     * 编解码器可能会调用此函数以执行几个独立的事情。
     * 它将在完成所有任务后才返回。
     * 用户可以使用一些多线程实现替换此函数，
     * 默认实现将按顺序执行这些部分。
     * @param count 要执行的事情的数量
     * - 编码：由 libavcodec 设置，用户可以覆盖。
     * - 解码：由 libavcodec 设置，用户可以覆盖。
     */
    int (*execute)(struct AVCodecContext *c, int (*func)(struct AVCodecContext *c2, void *arg), void *arg2, int *ret,
                   int count, int size);

    /**
     * 编解码器可能会调用此函数以执行几个独立的事情。
     * 它将在完成所有任务后才返回。
     * 用户可以使用一些多线程实现替换此函数，
     * 默认实现将按顺序执行这些部分。
     * @param c 也传递给 func 的上下文
     * @param count 要执行的事情的数量
     * @param arg2 不变地传递给 func 的参数
     * @param ret 执行的函数的返回值，必须为“count”个值留出空间。可以为 NULL。
     * @param func 将被调用 count 次的函数，jobnr 从 0 到 count-1。
     *             threadnr 将在 0 到 c->thread_count-1 < MAX_THREADS 的范围内，并且这样就不会
     *             两个同时执行的 func 实例具有相同的 threadnr。
     * @return 目前始终为 0，但代码应处理未来的改进，其中当任何对 func 的调用
     *         返回 < 0 时，不再允许进行对 func 的进一步调用，并且将返回 < 0。
     * - 编码：由 libavcodec 设置，用户可以覆盖。
     * - 解码：由 libavcodec 设置，用户可以覆盖。
     */
    int (*execute2)(struct AVCodecContext *c
            /**
             * 编解码器可能会调用此函数来执行多个独立的操作。
             * 它只会在完成所有任务后返回。
             * 用户可以将其替换为某些多线程实现，
             * 默认实现将按顺序执行这些部分。
             * @param c 传递给 func 的上下文
             * @param count 要执行的操作数量
             * @param arg2 不变地传递给 func 的参数
             * @param ret 已执行函数的返回值，必须有空间容纳 "count" 个值。可以为 NULL。
             * @param func 将被调用 count 次的函数，jobnr 从 0 到 count-1。
             *             threadnr 将在 0 到 c->thread_count-1 < MAX_THREADS 的范围内，并且这样两个在同一时间执行的 func 实例将不会有相同的 threadnr。
             * @return 目前始终为 0，但代码应处理将来的改进，其中当对 func 的任何调用返回 < 0 时，不再可以对 func 进行任何进一步的调用，并返回 < 0。
             * - 编码：由 libavcodec 设置，用户可以覆盖。
             * - 解码：由 libavcodec 设置，用户可以覆盖。
             */
                    int (*execute2)(struct AVCodecContext *c,
                                    int (*func)(struct AVCodecContext *c2, void *arg, int jobnr, int threadnr),
                                    void *arg2, int *ret, int count);

    /**
     * 针对 nsse 比较函数的噪声与 SSE 权重
     * - 编码：由用户设置。
     * - 解码：未使用
     */
    int nsse_weight;

    /**
     * 配置文件
     * - 编码：由用户设置。
     * - 解码：由 libavcodec 设置。
     */
    int profile;

    /**
     * 等级
     * - 编码：由用户设置。
     * - 解码：由 libavcodec 设置。
     * FF_LEVEL_UNKNOWN -99
     */
    int level;


    /**
     * 针对选定帧跳过循环滤波。
     * - 编码：未使用
     * - 解码：由用户设置。
     */
    enum AVDiscard skip_loop_filter;

    /**
     * 针对选定帧跳过 IDCT/反量化。
     * - 编码：未使用
     * - 解码：由用户设置。
     */
    enum AVDiscard skip_idct;

    /**
     * 针对选定帧跳过解码。
     * - 编码：未使用
     * - 解码：由用户设置。
     */
    enum AVDiscard skip_frame;

    /**
     * 包含文本字幕样式信息的标头。
     * 对于 SUBTITLE_ASS 字幕类型，它应该包含整个 ASS
     * [Script Info] 和 [V4+ Styles] 部分，以及 [Events] 行和
     * 以下的 Format 行。它不应包含任何 Dialogue 行。
     * - 编码：由用户设置/分配/释放（在 avcodec_open2() 之前）
     * - 解码：由 libavcodec 设置/分配/释放（由 avcodec_open2()）
     */
    uint8_t *subtitle_header;
    int subtitle_header_size;

    /**
     * 仅音频。编码器在音频开头插入的“预先”样本（填充）数量。即，解码后的前导样本的这一数量必须被调用者丢弃，以获取不包含前导填充的原始音频。
     *
     * - 解码：未使用
     * - 编码：由 libavcodec 设置。输出数据包上的时间戳由编码器调整，使其始终引用数据包中实际包含的第一个样本的时间戳，包括任何添加的填充。例如，如果时间基准为 1/采样率，并且第一个输入样本的时间戳为 0，则第一个输出数据包的时间戳将为 -initial_padding。
     */
    int initial_padding;

    /**
     * - 解码：对于在压缩比特流中存储帧速率值的编解码器，解码器可能会在此处导出它。当未知时为 { 0, 1}。
     * - 编码：可用于向编码器发出 CFR 内容的帧速率信号。
     */
    AVRational framerate;

    /**
     * 标称未加速像素格式，请参见 AV_PIX_FMT_xxx。
     * - 编码：未使用。
     * - 解码：由 libavcodec 在调用 get_format() 之前设置
     */
    enum AVPixelFormat sw_pix_fmt;

    /**
     * pkt_dts/pts 和 AVPacket.dts/pts 所在的时间基准。
     * - 编码未使用。
     * - 解码由用户设置。
     */
    AVRational pkt_timebase;

    /**
     * AVCodecDescriptor
     * - 编码：未使用。
     * - 解码：由 libavcodec 设置。
     */
    const AVCodecDescriptor *codec_descriptor;

    /**
     * PTS 校正的当前统计信息。
     * - 解码：由 libavcodec 维护和使用，并非旨在供用户应用程序使用
     * - 编码：未使用
     */
    int64_t pts_correction_num_faulty_pts; /// 到目前为止，不正确的 PTS 值的数量
    int64_t pts_correction_num_faulty_dts; /// 到目前为止，不正确的 DTS 值的数量
    int64_t pts_correction_last_pts;       /// 最后一帧的 PTS
    int64_t pts_correction_last_dts;       /// 最后一帧的 DTS

    /**
     * 输入字幕文件的字符编码。
     * - 解码：由用户设置
     * - 编码：未使用
     */
    char *sub_charenc;

    /**
     * 字幕字符编码模式。格式或编解码器可能会调整此设置（例如，如果它们自己进行转换）。
     * - 解码：由 libavcodec 设置
     * - 编码：未使用
     */
    int sub_charenc_mode;

    /**
     * 跳过处理 alpha（如果编解码器支持）。
     * 请注意，如果格式使用预乘 alpha（VP6 很常见，并且由于更好的视频质量/压缩而推荐）
     * 图像将看起来像 alpha 混合到黑色背景上。
     * 但是，对于不使用预乘 alpha 的格式
     * 可能会出现严重的伪像（尽管例如 libswscale 目前假设预乘 alpha）。
     *
     * - 解码：由用户设置
     * - 编码：未使用
     */
    int skip_alpha;

    /**
     * 在发生不连续性后要跳过的样本数量
     * - 解码：未使用
     * - 编码：由 libavcodec 设置
     */
    int seek_preroll;
    uint16_t *chroma_intra_matrix;//自定义内部量化矩阵 * 编码： 由用户设置，可以为空。
    uint8_t *dump_separator;//转储格式分隔符。  可以是", "或"\n "或其他任何东西
    char *codec_whitelist;//以','分隔的允许解码器列表。 如果为空，则允许所有解码器
    unsigned properties;//被解码的数据流的属性 编码：未使用解码：由 libavcodec 设置
    AVPacketSideData *coded_side_data;//与整个编码流相关的附加数据。 解码：未使用 编码：可由 libavcodec 在 avcodec_open2() 之后设置。
    int nb_coded_side_data;
    /**
     对描述输入（编码）*或输出（解码）帧的 AVHWFramesContext 的引用。
     * 或输出（解码）帧的引用。该引用由调用者设置，
     * 之后由 libavcodec 拥有（并释放）--调用者绝对不能读取该引用。
     * 设置后调用者不得读取。
     *
     * 解码： 该字段应由调用者通过 get_format() 设置。
     * 回调中设置。之前的引用（如果有的话）将总是
     * 在调用 get_format() 之前，libavcodec 将不对其进行解码。
     *
     * 如果使用默认的 get_buffer2()，并使用 hwaccel 像素
     * 格式，则将使用 AVHWFramesContext
     * 分配帧缓冲区。
     *
     * 编码： 对于配置为使用 hwaccel 像素*格式的硬件编码器，应设置此字段。
     * 格式的硬件编码器，调用者应将此字段设置为对 AVHWFramesContext 的引用。
     * 描述输入帧的 AVHWFramesContext 的引用。
     * AVHWFramesContext.format 必须等于
     * AVCodecContext.pix_fmt.
     *
     * 该字段应在调用 avcodec_open2() 之前设置。
     */
    AVBufferRef *hw_frames_ctx;
    int trailing_padding;//每个图像最大可接受的像素数。 解码：由用户设置 编码：由用户设置//
    int64_t max_pixels;//每个图像最大可接受的像素数。 解码：由用户设置 编码：由用户设置
    /**
     * 对 AVHWDeviceContext 的引用，描述将被硬件编码器/解码器使用的设备。
     * 硬件编码器/解码器使用的设备。 该引用由
     * 调用者设置，之后由 libavcodec 拥有（并释放）。
     *
     * 如果编解码器设备不需要
     * 如果编解码设备不需要硬件帧，或者使用的硬件帧将由
     * libavcodec 内部分配。 如果用户希望提供用作
     * 编码器输入或解码器输出时，则应使用 hw_frames_ctx。
     * 来代替。 如果在解码器的 get_format() 中设置了 hw_frames_ctx，则在解码时将忽略该字段。
     * 字段将在解码相关流段时被忽略，但
     * 但在调用另一次 get_format() 后，可在下一次调用中再次使用。
     *
     * 对于编码器和解码器，该字段都应在
     * 调用 avcodec_open2()之前设置，此后不得写入。
     *
     * 注意某些解码器可能要求初始化此字段，以支持 hw_file_codec_open2()
     * 在这种情况下，所有使用的帧上下文都必须在解码器上创建。
     * 使用的上下文必须在同一设备上创建。
     */
    AVBufferRef *hw_device_ctx;
    int hwaccel_flags;//AV_HWACCEL_FLAG_* 标志的位集，会影响硬件加速解码（如果激活）。编码：未使用，解码： 由用户设置（在 avcodec_open2() 之前或在 AVCodecContext.get_format 回调中）设置
    /**
      * 仅支持视频解码。某些视频编解码器支持裁剪，这意味着
      * 只显示解码帧的一个子矩形。 此
      * 选项控制 libavcodec 如何处理裁剪。
      *
      * 当设置为 1（默认值）时，libavcodec 将在内部应用裁剪。
      * 也就是说，它会修改输出帧的宽度/高度字段，并偏移
      * 数据指针（在保持对齐的情况下尽可能多地偏移，或
      * 如果设置了 AV_CODEC_FLAG_UNALIGNED 标志，则偏移全量），以便
      * 解码器输出的帧仅指裁剪过的区域。输出
      * 输出帧的 crop_* 字段将为零。
      *
      * 设置为 0 时，输出帧的宽度/高度字段将被设置为
      * 当设置为 0 时，输出帧的宽度/高度字段将被设置为编码尺寸，而 crop_* 字段将描述裁剪矩形。
      * 矩形。裁剪工作由调用者完成。
      *
      * 当使用不透明输出帧的硬件加速时、
      * libavcodec 无法从顶部/左边界进行裁剪。
      *
      * 注意：当此选项设置为零时，AVCodecContext 的宽度/高度字段将被设置为 "0"。
      * AVCodecContext 和输出 AVFrames 的宽度/高度字段具有不同的含义。编解码器
      * 上下文字段存储显示尺寸（编码的尺寸位于
      * 编码的宽度/高度），而帧字段存储编码的尺寸
      * 显示尺寸由 crop_* 字段决定）。
      */
    int apply_cropping;
    /*
     * 仅用于视频解码。 设置解码器分配给调用者使用的额外硬件帧数。
     * 解码器将分配给调用者使用的额外硬件帧数。 必须在调用
     * 必须在调用 avcodec_open2() 之前设置。
    */
    int extra_hw_frames;
    int discard_damaged_percentage; //用于丢弃帧的损坏样本百分比。 解码：用户设置，编码没有
    int64_t max_samples;//每帧最大接受的采样数。用户设置
    int export_side_data; //AV_CODEC_EXPORT_DATA_* 标志的位集，它影响着。元数据的类型
    int
    (*get_encode_buffer)(struct AVCodecContext *s, AVPacket *pkt, int flags);//编码： 由 libavcodec 设置，用户可以覆盖 。该回调必须是线程安全的
    AVChannelLayout ch_layout;//音频通道布局。编码：必须由调用者设置为 AVCodec.ch_layouts 之一。解码：可由调用者设置（如从容器中得知）。解码器可根据需要在解码过程中覆盖
    int64_t frame_num;//帧计数器，由 libavcodec 设置。 解码：目前从解码器返回的帧总数。编码：目前传递给编码器的帧总数。
} AVCodecContext;
```
