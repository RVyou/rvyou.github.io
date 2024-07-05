---
title: yolov8 安装、训练、标注
author: rvyou
date: 2024-04-19 10:11:51
categories: [yolo]
tags: [yolo]
---


## 安装相关依赖

> 这里安装一些依赖 [Windows 11 和 WSL2 安装 CUDA cuDNN OpenCV | rvyou](/posts/CUDA-cuDNN-OpenCV/)   
> 还有 python3.10

 pytorch 安装 https://pytorch.org/

ultralytics 和 标注工具 安装

```shell
pip install git+https://github.com/ultralytics/ultralytics.git@main
pip install labelimg
```

接下来就是用 yolo 相关命令[CLI - Ultralytics YOLO Docs](https://docs.ultralytics.com/usage/cli/#predict) 进行

## 训练自己数据集合

目录：

```shell
YOLO  目录名
├─ images  要标注图片
├─ labels   标注标签信息
├─ classes.txt  标签名字 每行一个
├─ yolov8.yaml  #https://github.com/ultralytics/ultralytics/blob/main/ultralytics/cfg/models/v8/yolov8.yamlhttps://github.com/ultralytics/ultralytics/blob/main/ultralytics/cfg/models/v8/yolov8.yaml
```

标注数据(在目录名打开命令行)

```shell
labelimg images classes.txt
```

标注完成后创建以下目录，把 labels、images 复制过去(train和valid比例是7:3 左右)

tips: 记得选生产是 yolo 的标注数据

```shell
├─images
├─labels
├─test
│  ├─images
│  └─labels
├─train   #训练数据 70%
│  ├─images
│  └─labels
└─valid  #验证数据 30%
    ├─images
    └─labels
```

创建 traffic.yaml

```vim
train: D:\yolotrain/images   #修改成train中images的绝对路径
val: D:\yolovalid/images     #修改成valid中images的绝对路径
test: D:\yolotest/images     #修改成test中images的绝对路径

nc: 1                                          #类的数目
names:                                        #每个类对应的名称
  0: mouse
```

修改yalov8.yaml

```vim
nc: 80 #改成自己的标签数量
```

## 使用 yolo

#### 训练：

```shell
yolo task=detect mode=train model=yolov8n.yaml data=traffic.yaml epochs=10 batch=3
#model：选择YOLOv8不同的模型配置文件，可选yolov8s.yaml、yolov8m.yaml等等 yolov8.yaml里面有说明
#data： 选择数据集的配置文件
#epochs：指的就是训练过程中整个数据集将被迭代多少次
#batch：一次看完多少张图片才进行权重更新
#从YAML构建一个新模型，并从头开始培训
#yolo detect train data=coco8.yaml model=yolov8n.yaml epochs=100 imgsz=640
#从预先训练的*.pt模型开始训练
#yolo detect train data=coco8.yaml model=yolov8n.pt epochs=100 imgsz=640
#从YAML建立一个新的模型，将预先训练的权重转移到它并开始训练
#yolo detect train data=coco8.yaml model=yolov8n.yaml pretrained=yolov8n.pt epochs=100 imgsz=640
```

训练好的模型会在  <u> runs/detect/train/weights/*.pt </u> 里面

一般使用 batch =1 进行递增训练正价准确度(有一定阈值)，相应epochs也要调大

#### 验证:

```shell
yolo task=detect mode=val model=runs/detect/train/weights/last.pt data=traffic.yaml batch=8
```


#### 测试：

```shell
yolo task=detect mode=predict model=runs/detect/train/weights/best.pt source=2.jpg
```
