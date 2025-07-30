---
title: 音视频(FFmpeg) - 3  ffprobe 源码
author: rvyou
date: 2024-09-07 19:33:53
categories: [audio_and_video ,FFmpeg]
tags: [audio_and_video,FFmpeg]
---

这里是 main 函数流程,删了一些代码
>> //******  这个开头的注释添加说明
```C
int main(int argc, char **argv)
{
    const AVTextFormatter *f;
    AVTextFormatContext *tctx;
    AVTextWriterContext *wctx;
    char *buf;
    char *f_name = NULL, *f_args = NULL;
    int ret, input_ret, i;
    init_dynload();
    setvbuf(stderr, NULL, _IONBF, 0); /* win32 runtime needs this */
    av_log_set_flags(AV_LOG_SKIP_REPEATED);

    options = real_options;
    parse_loglevel(argc, argv, options);
    avformat_network_init();
#if CONFIG_AVDEVICE
    avdevice_register_all();
#endif
    show_banner(argc, argv, options);
    //****** 解析 option
    ret = parse_options(NULL, argc, argv, options, opt_input_file);
    if (ret < 0) {
        ret = (ret == AVERROR_EXIT) ? 0 : ret;
        goto end;
    }

    if (do_show_log)
        av_log_set_callback(log_callback);

    /* mark things to show, based on -show_entries */
    SET_DO_SHOW(CHAPTERS, chapters);
   //****** 这里一堆 set

    if (do_bitexact && (do_show_program_version || do_show_library_versions)) {
        av_log(NULL, AV_LOG_ERROR,
               "-bitexact and -show_program_version or -show_library_versions "
               "options are incompatible\n");
        ret = AVERROR(EINVAL);
        goto end;
    }

    if (!output_format)
        output_format = av_strdup("default");
    if (!output_format) {
        ret = AVERROR(ENOMEM);
        goto end;
    }
    f_name = av_strtok(output_format, "=", &buf);
    if (!f_name) {
        av_log(NULL, AV_LOG_ERROR,
               "No name specified for the output format\n");
        ret = AVERROR(EINVAL);
        goto end;
    }
    f_args = buf;

    f = avtext_get_formatter_by_name(f_name);
    if (!f) {
        av_log(NULL, AV_LOG_ERROR, "Unknown output format with name '%s'\n", f_name);
        ret = AVERROR(EINVAL);
        goto end;
    }

    if (output_filename) {
        ret = avtextwriter_create_file(&wctx, output_filename);
    } else
        ret = avtextwriter_create_stdout(&wctx);

    if (ret < 0)
        goto end;

    AVTextFormatOptions tf_options = {
        .show_optional_fields = show_optional_fields,
        .show_value_unit = show_value_unit,
        .use_value_prefix = use_value_prefix,
        .use_byte_value_binary_prefix = use_byte_value_binary_prefix,
        .use_value_sexagesimal_format = use_value_sexagesimal_format,
    };

    if ((ret = avtext_context_open(&tctx, f, wctx, f_args, sections, FF_ARRAY_ELEMS(sections), tf_options, show_data_hash)) >= 0) {
        if (f == &avtextformatter_xml)
            tctx->string_validation_utf8_flags |= AV_UTF8_FLAG_EXCLUDE_XML_INVALID_CONTROL_CODES;

        avtext_print_section_header(tctx, NULL, SECTION_ID_ROOT);

        if (do_show_program_version)
            ffprobe_show_program_version(tctx);
        if (do_show_library_versions)
            ffprobe_show_library_versions(tctx);
        if (do_show_pixel_formats)
            ffprobe_show_pixel_formats(tctx);

        if (!input_filename &&
            ((do_show_format || do_show_programs || do_show_stream_groups || do_show_streams || do_show_chapters || do_show_packets || do_show_error) ||
             (!do_show_program_version && !do_show_library_versions && !do_show_pixel_formats))) {
            show_usage();
            av_log(NULL, AV_LOG_ERROR, "You have to specify one input file.\n");
            av_log(NULL, AV_LOG_ERROR, "Use -h to get full help or, even better, run 'man %s'.\n", program_name);
            ret = AVERROR(EINVAL);
        } else if (input_filename) {
            //****** 解析音视频
            ret = probe_file(tctx, input_filename, print_input_filename);
            if (ret < 0 && do_show_error)
                show_error(tctx, ret);
        }

        input_ret = ret;

        avtext_print_section_footer(tctx);

        ret = avtextwriter_context_close(&wctx);
        if (ret < 0)
            av_log(NULL, AV_LOG_ERROR, "Writing output failed (closing writer): %s\n", av_err2str(ret));

        ret = avtext_context_close(&tctx);
        if (ret < 0)
            av_log(NULL, AV_LOG_ERROR, "Writing output failed (closing formatter): %s\n", av_err2str(ret));

        ret = FFMIN(ret, input_ret);
    }

end:
    av_freep(&output_format);
    av_freep(&output_filename);
    av_freep(&input_filename);
    av_freep(&print_input_filename);
    av_freep(&read_intervals);

    uninit_opts();
    for (i = 0; i < FF_ARRAY_ELEMS(sections); i++)
        av_dict_free(&(sections[i].entries_to_show));

    avformat_network_deinit();

    return ret < 0;
}
```
## 参数解析
parse_options 这个方法做了参数解析

主要是 OptionDef real_options[] 这个变量，预设了 options 和对于处理方法(都是全局变量赋值)

### 解析音视频信息
probe_file

```c
static int probe_file(AVTextFormatContext *tfc, const char *filename,
                      const char *print_filename)
{
    InputFile ifile = { 0 };
    int ret, i;
    int section_id;

    do_analyze_frames = do_analyze_frames && do_show_streams;
    do_read_frames = do_show_frames || do_count_frames || do_analyze_frames;
    do_read_packets = do_show_packets || do_count_packets;

    ret = open_input_file(&ifile, filename, print_filename);
    if (ret < 0)
        goto end;

#define CHECK_END if (ret < 0) goto end

    nb_streams = ifile.fmt_ctx->nb_streams;
    REALLOCZ_ARRAY_STREAM(nb_streams_frames,0,ifile.fmt_ctx->nb_streams);
    REALLOCZ_ARRAY_STREAM(nb_streams_packets,0,ifile.fmt_ctx->nb_streams);
    REALLOCZ_ARRAY_STREAM(selected_streams,0,ifile.fmt_ctx->nb_streams);
    REALLOCZ_ARRAY_STREAM(streams_with_closed_captions,0,ifile.fmt_ctx->nb_streams);
    REALLOCZ_ARRAY_STREAM(streams_with_film_grain,0,ifile.fmt_ctx->nb_streams);

    for (i = 0; i < ifile.fmt_ctx->nb_streams; i++) {
        if (stream_specifier) {
            ret = avformat_match_stream_specifier(ifile.fmt_ctx,
                                                  ifile.fmt_ctx->streams[i],
                                                  stream_specifier);
            CHECK_END;
            else
                selected_streams[i] = ret;
            ret = 0;
        } else {
            selected_streams[i] = 1;
        }
        if (!selected_streams[i])
            ifile.fmt_ctx->streams[i]->discard = AVDISCARD_ALL;
    }

    if (do_read_frames || do_read_packets) {
        if (do_show_frames && do_show_packets &&
            tfc->formatter->flags & AV_TEXTFORMAT_FLAG_SUPPORTS_MIXED_ARRAY_CONTENT)
            section_id = SECTION_ID_PACKETS_AND_FRAMES;
        else if (do_show_packets && !do_show_frames)
            section_id = SECTION_ID_PACKETS;
        else // (!do_show_packets && do_show_frames)
            section_id = SECTION_ID_FRAMES;
        if (do_show_frames || do_show_packets)
            avtext_print_section_header(tfc, NULL, section_id);
        ret = read_packets(tfc, &ifile);
        if (do_show_frames || do_show_packets)
            avtext_print_section_footer(tfc);
        CHECK_END;
    }

    if (do_show_programs) {
        ret = show_programs(tfc, &ifile);
        CHECK_END;
    }

    if (do_show_stream_groups) {
        ret = show_stream_groups(tfc, &ifile);
        CHECK_END;
    }

    if (do_show_streams) {
        ret = show_streams(tfc, &ifile);
        CHECK_END;
    }
    if (do_show_chapters) {
        ret = show_chapters(tfc, &ifile);
        CHECK_END;
    }
    if (do_show_format) {
        ret = show_format(tfc, &ifile);
        CHECK_END;
    }

end:
    if (ifile.fmt_ctx)
        close_input_file(&ifile);
    av_freep(&nb_streams_frames);
    av_freep(&nb_streams_packets);
    av_freep(&selected_streams);
    av_freep(&streams_with_closed_captions);
    av_freep(&streams_with_film_grain);

    return ret;
}
```
## Reference
[github ffprobe](https://github.com/FFmpeg/FFmpeg/blob/master/fftools/ffprobe.c)


