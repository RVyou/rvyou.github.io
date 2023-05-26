---
title: mongoDB:unable to wirite wire message network
author: rvyou
date: 2023-04-26 19:16:00
categories: [kubernetes,Istio,Envoy]
tags: [kubernetes,Istio,Envoy,mongoDB,network]
---

### 起因

最近频繁出现mongoDB 写已经关闭的管道 日志

```shell
[ERROR] conection(1715277-12)unable to wirite wire message network: write tcp 10.311139:40938->17216.4875:27700:write broken pipe ### Deref 和 DerefMut trait
```

查看 mongoDB 当期连接，多出复数连接数，连接程序未知。连接隔一段时间会被 close 掉。

![Desktop View](/posts/20190808/mockup.png){: width="780" height="229" } _额外的连接_

![Desktop View](/posts/20190808/mockup.png){: width="977" height="322" } _连接隔一段时间会被 close 掉_

通过看客户端配置连接池是没有设置关闭时间的，怎么会被关闭呢，而且源码里面这种连接也没有心跳维持。

问题可能就出在 - 这个程序上了。会不会是 Istio 的呢。直接把 dev 环境的边车去掉后确实没有来。

[搜了一下看是不是有相关的问题](https://github.com/istio/istio/issues/24387)，果真有.... 

Envoy 的 TCPProxy 的 NetworkFilter 的 idle_timeout 参数，默认是 1h。

解决方案：

改客户端连接池配置 空闲 30分钟的连接自动断掉，Istio 配置就不该了
