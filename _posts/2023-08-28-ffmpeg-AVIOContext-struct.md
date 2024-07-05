---
title: ffmpeg 6.0 中 AVIOContext 结构体
author: rvyou
date: 2023-08-22 20:25:30
categories: [audio_and_video,ffmpeg,AVIOContext]
tags: [audio_and_video,ffmpeg,AVIOContext]
---
![Desktop View](assets/img/ffmpeg.png)
AVIOContext 是 ffmpeg 管理输入输出数据。

## AVIOContext

```c
typedef struct AVIOContext {
    /**
     * 私有选项类。
     * 
     * 如果此 AVIOContext 由 avio_open2() 创建，则会设置 av_class 并
     * 将选项传递给协议。
     * 
     * 如果此 AVIOContext 是手动分配的，则调用者可以设置 av_class。
     * 
     * 警告 - 此字段可能为 NULL，在这种情况下，请确保不要将此 AVIOContext
     * 传递给任何 av_opt_* 函数。
     */
    const AVClass *av_class;

    /*
     * 以下显示了 buffer、buf_ptr、
     * buf_ptr_max、buf_end、buf_size 和 pos 之间的关系，在读取和写入时
     * （因为 AVIOContext 用于两者）：
     * 
     **********************************************************************************
     *                                   读取
     **********************************************************************************
     * 
     *                            |              buffer_size              |
     *                            |---------------------------------------|
     *                            |                                       |
     * 
     *                         buffer          buf_ptr       buf_end
     *                            +---------------+-----------------------+
     *                            |/ / / / / / / /|/ / / / / / /|         |
     *  读取缓冲区:              |/ / 已使用 / | 待读取 /|         |
     *                            |/ / / / / / / /|/ / / / / / /|         |
     *                            +---------------+-----------------------+
     * 
     *                                                         pos
     *              +-------------------------------------------+-----------------+
     *  输入文件: |                                           |                 |
     *              +-------------------------------------------+-----------------+
     * 
     * 
     **********************************************************************************
     *                                   写入
     **********************************************************************************
     * 
     *                             |          buffer_size                 |
     *                             |--------------------------------------|
     *                             |                                      |
     * 
     *                                                buf_ptr_max
     *                          buffer                 (buf_ptr)       buf_end
     *                             +-----------------------+--------------+
     *                             |/ / / / / / / / / / / /|              |
     *  写入缓冲区:              | / / 待刷新 / / |              |
     *                             |/ / / / / / / / / / / /|              |
     *                             +-----------------------+--------------+
     *                               buf_ptr 可能位于此处
     *                               由于向后查找
     * 
     *                            pos
     *               +-------------+----------------------------------------------+
     *  输出文件: |             |                                              |
     *               +-------------+----------------------------------------------+
     * 
     */
    unsigned char *buffer;  /**< 缓冲区的开头。 */
    int buffer_size;        /**< 最大缓冲区大小 */
    unsigned char *buf_ptr; /**< 缓冲区中的当前位置 */
    unsigned char *buf_end; /**< 数据的结尾，可能小于
                                 buffer+buffer_size，如果读取函数返回
                                 请求的数据少于请求量，例如，对于尚未接收更多数据的流。 */
    void *opaque;           /**< 私有指针，传递给读取/写入/查找/...
                                 函数。 */
    int (*read_packet)(void *opaque, uint8_t *buf, int buf_size);
    int (*write_packet)(void *opaque, uint8_t *buf, int buf_size);
    int64_t (*seek)(void *opaque, int64_t offset, int whence);
    int64_t pos;            /**< 当前缓冲区在文件中的位置 */
    int eof_reached;        /**< 如果由于错误或文件结尾而无法读取，则为 true */
    int error;              /**< 包含错误代码，如果未发生错误，则为 0 */
    int write_flag;         /**< 如果打开以供写入，则为 true */
    int max_packet_size;
    int min_packet_size;    /**< 在刷新之前尝试至少缓冲这么多数据。 */
    unsigned long checksum;
    unsigned char *checksum_ptr;
    unsigned long (*update_checksum)(unsigned long checksum, const uint8_t *buf, unsigned int size);
    /**
     * 暂停或恢复网络流协议的播放 - 例如，MMS。
     */
    int (*read_pause)(void *opaque, int pause);
    /**
     * 使用指定 stream_index 将流中的时间戳定位到指定时间戳。
     * 一些不支持定位到字节位置的网络流协议需要此功能。
     */
    int64_t (*read_seek)(void *opaque, int stream_index,
                         int64_t timestamp, int flags);
    /**
     * AVIO_SEEKABLE_ 标志的组合，或者当流不可查找时为 0。
     */
    int seekable;

    /**
     * avio_read 和 avio_write 应尽可能直接满足
     * 而不是通过缓冲区，并且 avio_seek 将始终
     * 直接调用底层查找函数。
     */
    int direct;

    /**
     * 允许的协议的逗号分隔列表。
     */
    const char *protocol_whitelist;

    /**
     * 禁止的协议的逗号分隔列表。
     */
    const char *protocol_blacklist;

    /**
     * 用作 write_packet 的回调。
     */
    int (*write_data_type)(void *opaque, uint8_t *buf, int buf_size,
                           enum AVIODataMarkerType type, int64_t time);
    /**
     * 如果设置，不要分别为 AVIO_DATA_MARKER_BOUNDARY_POINT 调用 write_data_type，
     * 而是忽略它们并将它们视为 AVIO_DATA_MARKER_UNKNOWN（以避免从回调中返回不必要的小数据块）。
     */
    int ignore_boundary_point;

    /**
     * 写入缓冲区中向后查找之前的最大到达位置，
     * 用于跟踪已写入的数据，以便稍后刷新。
     */
    unsigned char *buf_ptr_max;

    /**
     * 此 AVIOContext 读取字节的只读统计信息。
     */
    int64_t bytes_read;

    /**
     * 此 AVIOContext 写入字节的只读统计信息。
     */
    int64_t bytes_written;
} AVIOContext;
```






