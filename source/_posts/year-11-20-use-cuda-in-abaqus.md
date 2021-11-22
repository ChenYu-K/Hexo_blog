---
title: 如何使用CUDA对abaqus进行加速
date: 2021-11-20 15:01:24
tags: ABAQUS
---

本文介绍了如何使用 [Nvidia CUDA](https://developer.nvidia.com/zh-cn/cuda-toolkit)[^1]的加速功能,使Abaqus计算加速。

<!--more-->

驱动都设置完成可直接查看查看 [环境变量设置](#环境变量设置)

# 安装显卡
---

提前查询好主板是否与显卡兼容，在购买显卡。
本次使用的是Nvidia 2021 新推出的 RTX A4000显卡[^2]。

|GPU特性|RTX A000|
|-----|------|
|GPU显存| 带纠错码ECC DDR6 16GB|
|显存带宽|448GB/s|
|图形总线|PCI-E X16|
|CUDA核心数|6144|
|单精度浮点计算|19.2 TFLOPS|

*具体可参考 [A4000规格书](https://www.nvidia.cn/content/dam/en-zz/Solutions/gtcs21/rtx-a4000/nvidia-rtx-a4000-datasheet.pdf)

1. 插入卡槽
2. 连接显卡电源（6Pin）
3. 开机测试

# 安装显卡驱动
---

在[Nvida显卡驱动官网](https://www.nvidia.co.jp/Download/index.aspx?lang=jp)找到合适自己的显卡驱动下载，并重启。
![](https://i.imgur.com/ylSk04C.png)

# 安装CUDA 工具
---

[CUDA工具包安装地址](https://developer.nvidia.com/zh-cn/cuda-toolkit)

![](https://i.imgur.com/KgAsN7K.png)


CUDA的安装需要较长时间，属于正常情况。


# 环境变量设置
---

在这里有两种方法可以开启CUDA的加速

## 直接编辑系统环境变量，如下图

在系统全局变量里加入

|环境变量|内容|
|------|------|
|变量名|`ABA_ACCELERATOR_TYPE`|
|值|`PLATFORM_CUDA`|

![](https://i.imgur.com/xEUpwAm.png)

## 编辑`abaqus_v6.env`  

 在`abaqus_v6.env`[^3]文件的句末加上
 ```TEXT
os.environ["ABA_ACCELERATOR_TYPE"]="PLATFORM_CUDA" # Nvidia
 ```
 的字段使其可以使用CUDA加速工具加速ABAQUS。  



![](https://i.imgur.com/6Ui3AIK.png)

# 是否加速成功

---

成功加速Abaqus反馈的Log里面会出现如下加速成功的字符。
![](https://i.imgur.com/WZRgoEt.png)

# Reference

[^1]: Nvidia Cuda 适用于类似于`RTX` or `Quardo`之类的 计算卡，一般的 `Geforce` 显卡不不推荐使用CUDA加速。  
[^2]: [官网介绍](https://www.nvidia.com/en-us/design-visualization/rtx-a4000/) ,购入金额15万日元。  
[^3]: `abaqus_v6.env`的路径一般是在`C:\SIMULIA\EstProducts\2020\win_b64\SMA\site`这里。
