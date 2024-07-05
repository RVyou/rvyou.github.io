---
title: ffmpeg 6.0 中 AVStream 结构体
author: rvyou
date: 2023-08-22 20:25:30
categories: [audio_and_video,ffmpeg,AVStream]
tags: [audio_and_video,ffmpeg,AVStream]
---
![Desktop View](assets/img/ffmpeg.png)
AVStream 是 ffmpeg 用于描述多媒体流。

## AVStream

```c
typedef struct AVStream {
    /**
     * @ref avoptions 的类。在流创建时设置。
     */
    const AVClass *av_class;

    int index;    /**< AVFormatContext 中的流索引 */
    /**
     * 格式特定的流 ID。
     * 解码：由 libavformat 设置
     * 编码：由用户设置，如果未设置则由 libavformat 替换
     */
    int id;

    /**
     * 与此流关联的编解码器参数。由
     * libavformat 在 avformat_new_stream() 和 avformat_free_context() 中分配和释放
     * 分别。
     * 
     * - 解复用：在流创建时或在
     *             avformat_find_stream_info() 中由 libavformat 填充
     * - 复用：在 avformat_write_header() 之前由调用者填充
     */
    AVCodecParameters *codecpar;

    void *priv_data;

    /**
     * 这是时间的基本单位（以秒为单位），以其
     * 帧时间戳表示。
     * 
     * 解码：由 libavformat 设置
     * 编码：可以在 avformat_write_header() 之前由调用者设置，以
     *           向复用器提供有关所需时间基准的提示。在
     *           avformat_write_header() 中，复用器将覆盖此字段
     *           使用将实际用于写入文件的时间戳的时间基准
     *           （这可能与用户提供的时间基准有关，也可能无关，具体取决于格式）。
     */
    AVRational time_base;

    /**
     * 解码：流中第一帧的 pts，以呈现顺序排列，以流时间基准为单位。
     * 仅在您绝对 100% 确定您设置的值时才设置此值
     * 它确实是第一帧的 pts。
     * 这可能是未定义的（AV_NOPTS_VALUE）。
     * @注意 ASF 标头不包含正确的时间戳，ASF
     * 解复用器不得设置此值。
     */
    int64_t start_time;

    /**
     * 解码：流的持续时间，以流时间基准为单位。
     * 如果源文件没有指定持续时间，但指定了
     * 比特率，此值将根据比特率和文件大小估算。
     * 
     * 编码：可以在 avformat_write_header() 之前由调用者设置，以
     * 向复用器提供有关估计持续时间的提示。
     */
    int64_t duration;

    int64_t nb_frames;                 ///< 如果已知，则此流中的帧数，否则为 0

    /**
     * 流的处置 - AV_DISPOSITION_* 标志的组合。
     * - 解复用：在创建流或在
     *             avformat_find_stream_info() 中由 libavformat 设置。
     * - 复用：可以在 avformat_write_header() 之前由调用者设置。
     */
    int disposition;

    enum AVDiscard discard; ///< 选择哪些数据包可以随意丢弃，无需解复用。

    /**
     * 样本纵横比（如果未知则为 0）
     * - 编码：由用户设置。
     * - 解码：由 libavformat 设置。
     */
    AVRational sample_aspect_ratio;

    AVDictionary *metadata;

    /**
     * 平均帧速率
     * 
     * - 解复用：可以在创建流或在
     *             avformat_find_stream_info() 中由 libavformat 设置。
     * - 复用：可以在 avformat_write_header() 之前由调用者设置。
     */
    AVRational avg_frame_rate;

    /**
     * 对于具有 AV_DISPOSITION_ATTACHED_PIC 处置的流，此数据包
     * 将包含附加的图片。
     * 
     * 解码：由 libavformat 设置，调用者不得修改。
     * 编码：未使用
     */
    AVPacket attached_pic;

    /**
     * 应用于整个流的侧边数据的数组（即，容器不允许它在数据包之间更改）。
     * 
     * 此数组中的侧边数据与数据包中的侧边数据之间可能没有重叠。即，给定的侧边数据要么由复用器导出
     * （解复用）/ 由调用者设置（复用）在此数组中，然后它永远不会
     * 出现在数据包中，或者侧边数据通过
     * 数据包导出/发送（始终在值变为已知或
     * 更改的第一个数据包中），然后它不会出现在此数组中。
     * 
     * - 解复用：在创建流时由 libavformat 设置。
     * - 复用：可以在 avformat_write_header() 之前由调用者设置。
     * 
     * 由 libavformat 在 avformat_free_context() 中释放。
     * 
     * @see av_format_inject_global_side_data()
     */
    AVPacketSideData *side_data;
    /**
     * AVStream.side_data 数组中的元素数量。
     */
    int            nb_side_data;

    /**
     * 指示在流上发生的事件的标志，AVSTREAM_EVENT_FLAG_ 的组合。
     * 
     * - 解复用：可能由解复用器在 avformat_open_input()、
     *   avformat_find_stream_info() 和 av_read_frame() 中设置。标志必须清除
     *   由用户清除，一旦事件已处理。
     * - 复用：可以在 avformat_write_header() 之后由用户设置。以
     *   指示用户触发的事件。复用器将清除已处理事件的标志
     *   在 av_[interleaved]_write_frame() 中。
     */
    int event_flags;
/**
 * - 解复用：解复用器从文件中读取了新的元数据，并相应地更新了
 *     AVStream.metadata
 * - 复用：用户更新了 AVStream.metadata，并希望复用器将其写入
 *     到文件中
 */
#define AVSTREAM_EVENT_FLAG_METADATA_UPDATED 0x0001
/**
 * - 解复用：从文件中读取了此流的新数据包。这
 *   事件仅供参考，并不保证新数据包
 *   此流必须从 av_read_frame() 返回。
 */
#define AVSTREAM_EVENT_FLAG_NEW_PACKETS (1 << 1)

    /**
     * 流的真实基本帧速率。
     * 这是可以准确表示所有时间戳的最低帧速率（它是所有时间戳的最小公倍数
     * 流中的帧速率）。注意，此值只是一个猜测！
     * 例如，如果时间基准为 1/90000，所有帧都具有以下任一值
     * 大约 3600 或 1800 个计时器滴答，则 r_frame_rate 将为 50/1。
     */
    AVRational r_frame_rate;

    /**
     * 时间戳中的位数。用于控制包装。
     * 
     * - 解复用：由 libavformat 设置
     * - 复用：由 libavformat 设置
     * 
     */
    int pts_wrap_bits;
} AVStream;
```




