---
title: FFmpeg 6.0 中 AVFormatContext 结构体
author: rvyou
date: 2023-08-22 19:52:30
categories: [audio_and_video,FFmpeg,AVFormatContext]
tags: [audio_and_video,FFmpeg,AVFormatContext]
---
![Desktop View](assets/img/ffmpeg.png)
AVFormatContext 是 FFmpeg 比较核心的结构，贯彻处理核心内容，毕竟是上下文。

## AVFormatContext

```c
typedef struct AVFormatContext {
    const AVClass *av_class;                //AVClass用于打印log和AVOption的引用，由avformat_alloc_context() 函数创建
    const struct AVInputFormat *iformat;    //输入文件容器格式，大部分是函数指针
    const struct AVOutputFormat *oformat;   //输出文件容器格式，大部分是函数指针
    //格式化私有数据，format/oformat不为空时，用支持AVOptions的结构
    //封装时 avformat_write_header 设置 oformat->priv_data，占用内存空间长度由oformat->priv_data_size
    //解封 avformat_open_input 设置 iformat->priv_data
    void *priv_data;                        
    AVIOContext *pb;                        //管理输入输出数据的结构体
    //标记信号流属性。AVFMTCTX_*的集合。该字段的设置基本上都在各个解封装器的read_header方法中，例如flv_read_header函数。
    //AVFMTCTX_NOHEADER 表示不存在header（流是动态添加的）
    //AVFMTCTX_UNSEEKABLE 表示流绝对不可定位，尝试调用seek函数将失败。 例如 HLS 可能在运行时动态更改
    int ctx_flags;
    unsigned int nb_streams;                //音视频流的个数(视频、音频、字幕等)
    AVStream **streams;                     //音视频流
    char *url;                              //文件名 或url路径
    int64_t start_time;                     //一般没用以streams为准，AVSTREAM 推导出来，在解封装的时候有意义，由libavformat模块赋值。
    int64_t duration;                       //一般没用以streams为准，时长,单位：微秒us，转换为秒需要除以1000000
    int64_t bit_rate;                       //一般没用以streams为准，比特率（单位bps，转换为kbps需要除以1000）
    unsigned int packet_size;               //数据包大小和下面 ma_delay 是不是流时候使用
    int max_delay;
    int flags;                              //标记(源码有define定义) avformat_open_input / avformat_write_header 
    int64_t probesize;                      //从输入流探测数据包的大小 byte 在avformat_open_input之前设置 默认5000000
    int64_t max_analyze_duration;           //最大的分析数据的时长 微秒，在检测编码格式时使用，在avformat_find_stream_info前设置
    const uint8_t *key;                     //关键字
    int keylen;
    unsigned int nb_programs;               //
    AVProgram **programs;                   //指向实际节目的指针%
    enum AVCodecID video_codec_id;          //强制视频编解码器 ID。解复用： 由用户设置
    enum AVCodecID audio_codec_id;
    enum AVCodecID subtitle_codec_id;
    unsigned int max_index_size;            //每个数据流索引的最大内存容量（以字节为单位），超过截断
    unsigned int max_picture_buffer;        //用于缓冲从实时捕捉设备获取的帧的最大内存容量（以字节为单位）。
    unsigned int nb_chapters;               
    AVChapter **chapters;                   //指向章节数据的指针%
    AVDictionary *metadata;                 //文件元数据
    int64_t start_time_realtime;            //码流开始的实际时间，以毫秒为单位
    int fps_probe_size;                     //表明使用了多少帧用于格式检测
    int error_recognition;                  //错误检测的阈值
    AVIOInterruptCB interrupt_callback;     //IO层用户自定义的回调结构
    int debug;
    int64_t max_interleave_delta;           //交错的最长缓冲时间 av_interleaved_write_frame() 将等待每个数据流至少有一个数据包后，才会将任何数据包实际写入输出文件。
    int strict_std_compliance;
    int event_flags;                        //文件中发生的事件的标志，avfmt_event_flag_*。  avformat_open_input avformat_find_stream_info av_read_frame 设置， 事件处理完成后，用户必须清除标记。
    int max_ts_probe;                       //在等待第一个时间戳时要读取的最大数据包数量。仅限解码。
    int avoid_negative_ts;                  //在多路复用过程中避免负时间戳。 AVFMT_AVOID_NEG_TS_ 使用 av_interleaved_write_frame() 时效果更好
    int ts_id;                              //传输流的ID
    int audio_preload;                      //以微秒为单位的音频预载
    int max_chunk_duration;                 //以微秒为单位的最大分块时间
    int max_chunk_size;                     //以字节为单位的最大块大小
    int use_wallclock_as_timestamps;        //强制使用挂钟时间戳作为数据包的 pts/dts * 如果存在 B 帧，则会产生未定义的结果。
    int avio_flags;                         //avio 标志，用于强制 AVIO_FLAG_DIRECT
    enum AVDurationEstimationMethod duration_estimation_method; // 持续时间字段可通过多种方式估算，该字段可用于  了解持续时间的估算方式。  编码：未使用  - 解码： 由用户读取
    int64_t skip_initial_bytes;             //打开数据流时跳过初始字节
    unsigned int correct_ts_overflow;       // 纠正单个时间戳溢出
    int seek2any;                           //强制查找任何（也包括非关键帧）帧
    int flush_packets;                      //在每个数据包之后刷新 I/O 上下文 编码
    int probe_score;                        //格式探测得分 0-100
    int format_probesize;                   //从输入中读取的最大字节数，以便识别 ref AVInputFormat "输入格式"。仅在格式未被调用者明确设置时使用
    char *codec_whitelist;                  //以','分隔的允许解码器列表
    char *format_whitelist;                 //以','分隔的允许使用的解码器列表
    int io_repositioned;                    //IO 重新定位标志
    const AVCodec *video_codec;
    const AVCodec *audio_codec;
    const AVCodec *subtitle_codec;
    const AVCodec *data_codec;
    int metadata_header_padding;
    void *opaque;                           //用户数据。 这里存放用户的一些私人数据。
    av_format_control_message control_message_cb;
    int64_t output_ts_offset;               //输出时间戳偏移，以微秒为单位。
    uint8_t *dump_separator;                //转储格式分隔符。可以是", "或"\n "或其他任何东西
    enum AVCodecID data_codec_id;           //强制数据 codec_id。
    char *protocol_whitelist;               // 以','分隔的允许协议列表。
    int (*io_open)(struct AVFormatContext *s, AVIOContext **pb, const char *url,
                   int flags, AVDictionary **options);   //用于打开新 IO 流的回调。
    void (*io_close)(struct AVFormatContext *s, AVIOContext *pb); //已过时，请使用 io_close2
    char *protocol_blacklist;               //',' separated list of disallowed protocols.
    int max_streams;                        //流的最大数量
    int skip_estimate_duration_from_pts;    //跳过estimate_timings_from_pts中的持续时间计算
    int max_probe_packets;                  //可探测的最大数据包数量
    int (*io_close2)(struct AVFormatContext *s, AVIOContext *pb); //关闭使用 AVFormatContext.io_open() 打开的数据流的回调。
} AVFormatContext;
```
