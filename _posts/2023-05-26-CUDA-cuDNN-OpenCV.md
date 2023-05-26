---
title: Windows 11 和 WSL2 安装 CUDA cuDNN OpenCV
author: rvyou
date: 2023-05-26 17:32:50+08:00
categories: [AI,install]
tags: [AI,WSL2,Windows,CUDA,cuDNN,OpenCV]
---

## windows 安装

### 安装 CUDA

[CUDA Toolkit Archive  NVIDIA Developer](https://developer.nvidia.com/cuda-toolkit-archive)

 我这里下载 cudn 12

ps: c c++ windows

> [Visual Studio 较旧的下载 - 2019、2017、2015 和以前的版本 (microsoft.com)](https://visualstudio.microsoft.com/zh-hans/vs/older-downloads/)
>
> [Visual Studio 2022 IDE - 适用于软件开发人员的编程工具 (microsoft.com)](https://visualstudio.microsoft.com/zh-hans/vs/)

### 安装 cuDNN

[cuDNN Archive  NVIDIA Developer](https://developer.nvidia.com/rdp/cudnn-archive)

### 环境变量配置

```
CUDA_PATH = C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1
CUDA_PATH_V12_1 = C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1
```

### 下载 zlibwapi.dll

[Installation Guide NVIDIA Docs](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html#install-zlib-windows)

放到c:\windows\system32下面

### 安装 CMake

[Download  CMake](https://cmake.org/download/)

这里安装是 3.26

### 安装 openCV

[opencv_contrib](https://github.com/opencv/opencv_contrib/tags)

[opencv](https://github.com/opencv/opencv/releases)

这里下载是 4.7.0

- cmake-gui.exe 打开配置项目目录是 opencv 所在目录，安装目录自己选着一个

- 第一次点击 **configure**  默认就行，点击Finish，第二次点击 **configure**，开始编译了

- configure done 后

  - search 框内输入 cuda，三个全部 value 打勾

  - search 框搜 MODULES，在 OPENCV_EXTRA_MODULES_RATH 一项，添加opencv_contrib4.5.1中的modules目录(最好用过 按钮 ... 添加)

  - search 框搜 NON， OPENCV_ENABLE_NONFREE value 打勾

  - search 框搜 world，build_opencv_world 的 value 打勾

  - search 框搜 test ，BUILD_TESTS ， BUILD_PERF_TESTS ，BUILD_EXAMPLES，去掉打勾

- 第三次点击 **configure**

- search 框内输入 cuda，CUDA_ARCH_BIN 自己显卡的算力 [算力查询](https://developer.nvidia.com/cuda-gpus)

- 第四次点击 **confige**，然后点击 **Generate**，然后点击 **Open Project**，自动启动你的**Visual Studio**

- 选择 Release x64，右边解决方案资源管理器 CMakeTargets 下的 ALL_BUILD，右键`生成`

- 解决方案资源管理器  CMakeTargets 下的 INSTALL 右键  `生成`

- 安装目录 出现一个 install 文件夹

### 环境变量配置

```
OPENCV_LINK_LIBS = opencv_world470,opencv_img_hash470
OPENCV_INCLUDE_PATHS = D:\App\opencv\install\include
OPENCV_LINK_PATHS = D:\App\opencv\install\x64\vc17\lib
```

ps:

> opencv_img_hash470 比较特殊，以前版本是在 opencv_world 里面的，现在独立出来了

## WSL2 安装

更上面一样对应WSL版本，gcc 可能需要更高级版本

`CUDA`： Linux -> x86_64 ->WSL-Ubuntu -> 2.0 -> 看自己喜欢那种方式

`cuDNN`: 安装对应ubuntu

WSL2 屏蔽 Windows PATH 方案 ： /`etc/wsl.conf` 中，设置`appendWindowsPath=false`

### Reference

[Linking error on Windows 10 · Issue #360 · twistedfall/opencv-rust · GitHub](https://github.com/twistedfall/opencv-rust/issues/360)

[Installation Guide - NVIDIA Docs](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html)
