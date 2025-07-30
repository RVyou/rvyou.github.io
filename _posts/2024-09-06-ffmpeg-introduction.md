---
title: 音视频(FFmpeg) - 2  FFmpeg 相关
author: rvyou
date: 2024-09-06 19:25:25
categories: [audio_and_video ,FFmpeg]
tags: [audio_and_video,FFmpeg]
---

## ffmpeg 命令
这是一些一些常用的[命令](https://ffmpeg.org/ffmpeg.html)

>stream_type ：'v'或'V'表示视频，'a'表示音频，'s'表示字幕，'d'表示数据，'t'表示附件<br>
>stream_index ：从0开始
如果不指定 map ， ffmpeg 会自动选择各种流中最大的。
> - 一般是 输入->解码->过滤->编码->输出 的流程
>
> 高版本有的打印输出流程内容：
> - -print_graphs_file  输出到指定的文件 
> - -print_graphs_format  指定输出格式，包括compact, csv, flat, ini, json, xml, mermaid, mermaidhtml，默认json

| 命令                                       | 说明                                                                                        |
|------------------------------------------|-------------------------------------------------------------------------------------------|
| -r 25                                    | 帧数设置强制性，例子：让原视频的一帧编程24帧进行输出 》  ffmpeg -r 1 -i input.m2v -r 24 output.mp4                  |
| -c  copy 或 -codec copy	                  | 编码这样写默认全部都copy  或者指定编码和码流 -c:v libx264 -c:a copy  格式  -codec:[stream_type]:[stream_index] |
| -vn、-an 、-sn 、-dn                        | 跳过视频、跳过音频、跳过字幕、跳过数据流                                                                      |
| -t   1                                   | 输出文件音视频时间                                                                                 |
| -f s16le                                 | 是 format 的缩写，是强制指定输入或输出的格式 ，                                                              |    
| -ar 44100                                | 采样率                                                                                       |   
| -ac 2                                    | 通道                                                                                        |   
| -q 1                                     | 质量 图片或者音频质量                                                                               |  
| -to  55	                                 | 跳到到  ‘12:03:45’或者 秒  和 -t 互斥，且 -t 优先                                                      |
| -ss  55 	                                | 从 ‘12:03:45’或者 秒 秒开始输出                                                                    |
| -frames:v  1                             | 设置输出视频帧的数量                                                                                |
| -discard (input)                         | 丢弃某些帧 bidir 双向参考帧(B)、nokey 关键帧（I）、noref（丢弃所有非参考帧）、all                                     |
| -filter_complex                          | 过滤器  别名 -lavfi                                                                            |
| vf 和 -af                                 | 分别作为 -filter:v （视频）和 -filter:a （音频）的别名                                                    |
| -metadata[:metadata_specifier] key=value | 设置源数据 :metadata_specifier 可以指定流                                                           |
| 查看类全局命令                                  |                                                                                           |
| -formats	                                | 	显示可用格式（包括设备）。                                                                            |
| -demuxers	                               | 显示可用解复用器。                                                                                 |
| -muxers		                                | 显示可用复用器。                                                                                  |
| -devices	                                | 	显示可用设备。                                                                                  |
| -codecs   	                              | 	显示 libavcodec 所知的所有编解码器。                                                                 |
| -decoders  	                             | 显示可用解码器。                                                                                  |
| -encoders	                               | 	显示所有可用的编码器。                                                                              |
| -bsfs		                                  | 	显示可用的位流过滤器。                                                                              |
| -protocols	                              | 	显示可用协议。                                                                                  |
| -filters	                                | 	显示可用的 libavfilter 滤镜。                                                                    |
| -pix_fmts	                               | 	显示可用的像素格式。                                                                               |
| -sample_fmts                             | 	显示可用的样本格式。                                                                               |
| -layouts	                                | 	显示通道名称和标准通道布局。                                                                           |
| -dispositions                            | 	显示流配置。                                                                                   |


- 滤镜 
滤镜太多了，这里只是大概说一下有哪些常用滤镜。其他还是要看[官方文档](https://ffmpeg.org/ffmpeg-filters.html)
> **:**是连接参数，**;** 滤镜结束下一条滤镜执行。**[xx]** 或者 **[0:v]**别名作为输出或者输入 

音频滤镜：
- afftdn 降噪
- amix  混音
- volume 音量
- anullsrc 空音频源
- amerge 混多通道的声音
- 其他都是进行修改声音进行 分贝修改、高低音修改、消除、优化、特效等等

视频滤镜:
- addroi  特定区域高质量编码
- backgroundkey  将静态背景变为透明。
- blend  将两个视频帧混合在一起。
- chromahold 删除除特定颜色之外的所有颜色的所有颜色信息。
- scale 调整宽高
- overlay 叠加2个视频
- colorkey 指定某个颜色相似度半径内的像素的 alpha 分量设置为 0
- colorhold 删除出谋个特定颜色之外所有颜色
- concat   连接音视频流
- overlay 叠加前景
- crop  裁剪为给定的尺寸
- cropdetect  自适应裁剪给定尺寸
- despill	去除由绿幕或蓝幕反射颜色引起的前景颜色不必要的污染
- drawbox  画框
- fade  淡出淡入效果
- framestep 每n帧选择一帧
- hflip 水平翻转
- hstack 水平堆叠
- hue 修改饱和度
- maskfun 从输入视频创建蒙版
- blend	混合输出2个视频  
- mix 视频混合
- 其他都是进行图像、颜色、yue、滤镜算法、编码 、神经网络等等相关操作。


## 链接库相关
- avcodec
  - 音视频编解码器,还包含了一些字幕编解码功能
- avdevice
  - 提供访问操作系统底层音视频输入/输出设备的接口 
- avfilter 
  - 提供音视频滤镜 (filter) 功能，对音视频流进行各种处理和修改
    - 视频滤镜: 缩放 (scaling)、裁剪 (cropping)、旋转 (rotating)、叠加 (overlaying)、颜色校正、模糊、锐化等。
    - 音频滤镜: 音量调节、声道混合、均衡器、变速不变调等
    - 滤镜图 (filtergraph): 通过 滤镜图 串联起来可以实现复杂的处理流程
- avformat 
  - 处理各种多媒体容器格式 (container formats) 的封装 (muxing) 和解封装 (demuxing)，还有网络协议
- avutil
  - 一个核心工具库，各种通用函数和数据结构。
- postproc
  - 供视频后期处理功能。 主要用于一些旧的或特定的视频后处理效果，如去块效应 (deblocking)、去环效应 (deringing) 等，用于改善解码后视频的质量。现在很多这类功能已经被 `avfilter` 或编解码器内部处理所取代或增强
- swresample 
  - 提供高质量的音频重采样 (resampling)、声道布局转换 (channel layout conversion) 和采样格式转换 (sample format conversion) 功能。
- swscale
  - 提供高质量的图像缩放 (scaling) 和颜色空间/像素格式转换 (color space/pixel format conversion) 功能
  
## ffmpeg 代码指南

### 常用结构说明
ffmpeg一般上下文有
- AVFormatContext : I/O 上下文 ; ffmepg 使用当中贯穿全局。部分属性需要使用 avformat_find_stream_info 嗅探才知道
- AVPacket : 管理压缩后的媒体数据的结构，不包含数据只是引用了数据。一帧可能音频也可能是视频。av_packet_unref 
每次读完一帧使用， av_packet_free 不使用这个结构后释放
- AVCodecContext : 编码器  解码器 上下文
- AVCodec ： 编解码信息
- AVCodecParameters ：编解码参数
- AVFrame ：解码之后的 YUV 数据，不包含数据只是引用了数据。



## Reference
[罗上文-ffpmpeg原理](https://ffmpeg.xianwaizhiyin.net/compile-ffmpeg/nvidia.html)


