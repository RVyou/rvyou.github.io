---
title: windows 驱动介绍和开发工具安装
author: rvyou
date: 2024-11-26 14:26:00
categories: [windows,driver]
tags: [windows,driver]
---

##  windows驱动
| 类型                                | 说明                                                | 
|-----------------------------------|---------------------------------------------------| 
| WDM（Windows Driver Model）         | Windows98和Windows2000的驱动程序设计规范                    |
| DDK（Driver Developer Kit）         | 2000/XP/2003的驱动开发                                 |
| WDF（Windows Driver Foundation）    | WDF 比 WDM更先进、合理，WDF和WDM的关系有点类似于MFC和Windows SDK的关系 |
| WDK（Windows Driver Kit）           | WDK可以看做是DDK的升级版本                                             |
| KMDF（Kernel-Mode Driver Framework） | 作为内核模式操作系统组件的一部分执行，它们管理I/O、即插即用、内存、进程和线程、安全等。内核模式驱动程序通常为分层结构。KMDF是Windows系统底层驱动，文件名为：*.SYS，Vista为2万多外设提供了KMDF，其中也包括USB2.0，因此对于具有USB2.0协议的FX2，只需编写与FX2相关的UMDF即可。 |
| UMDF（User-Mode Driver Framework）  |  UMDF是用户层驱动，文件名为：*.DLL。用户模式驱动程序支持基于协议或基于串行总线（如摄像机和便携音乐播放器）的设备。                                            |


### KMDF（Kernel-Mode Driver Framework）
#### 1.可参阅 MSDN中“Getting Started with Kernel-Mode Driver Framework ”

### UMDF（User-Mode Driver Framework）
####
关于KMDF更多的内容，可参阅 MSDN中“ Introduction to UMDF“。
## Visual Studio 安装 支持驱动开发 
[安装教程微软官方](https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk)

tp: 使用 管理员 权限进行 安装 Visual Studio 扩展，`Windows 驱动程序工具包` 可能是英文 `Windows Driver Kit`
未完成
## Reference

