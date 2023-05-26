---
title: k8s pod OOM
author: rvyou
date: 2022-12-14 11:30:52
categories: [kubernetes,excessive_memory_usage,golang]
tags: [kubernetes,excessive_memory_usage,golang]
---

最近线上一个视频服务出现 OOM (4个 pod 其中一个) ,这个服务本身什么有 pprof 。

那时候感觉是 BOS SDK 的问题，也看了一下 SDK 的源码。发现内存有 copy 不判断大小直接 copy 来计算大小和 md5 (看 api 说明只需要知道长度就能上传，copy 只是为了计算长度和 md5。直接修改 SDK 代码，一般情况是知道长度，无法获取长度在用循环读取方式计算)。也给服务挂上 pprof 。



重新部署上线后 发现还是有一个 pod 异常，内存占用很高(第一感觉内存泄漏？)....

进入容器内部查看实际占用只有 200M 左右( cat /proc/1/status )这就很奇怪，pprof 看起来也很正常。

于是乎感觉是使用内存计算不一样？

google 一下发现确实是这样 cache 使用也被算上了(但是也不应该调整 Linux 策略)。但是还是不太多只有一个 pod 异常。

那这是变成负载均衡策略问题(轮询)，统计ELK 日志发现确实视频接收时候有可能大部分落在一个节点上( istio 再添加针对单个接口的负载均衡策略)

ps:上面这个搞了几天，虽然问题不大。但是定位很崎岖...
