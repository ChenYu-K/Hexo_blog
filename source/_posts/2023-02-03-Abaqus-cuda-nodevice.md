---
title: Can't run abaqus in ubuntu Ubuntu 22.04.1 LTS with GPU(CUDA) accelerator. Initializing the CUDA Driver NO_DEVICE
date: 2023-02-03 15:01:24
tags: ABAQUS
toc: true
---

本文介绍了在Linux下运行Abaqus并调用CUDA时，出现`Error initializing the CUDA Driver NO_DEVICE`的解决方案。

English ver:
**Can't run abaqus in ubuntu Ubuntu 22.04.1 LTS with GPU(CUDA) accelerator. Initializing the CUDA Driver NO_DEVICE**
You can view this site, is my anser.

https://askubuntu.com/questions/1449122/cant-run-abaqus-in-ubuntu-ubuntu-22-04-1-lts-with-gpucuda-accelerator-initia

<!--more-->

# 问题描述
当在Linux下运行abaqus job=jobname cpus=4 gpus=1 int 时，调用CUDA加速时出现以下错误：
```
USING ACCELERATOR PLATFORM_CUDA
Error initializing the CUDA Driver NO_DEVICE
WARNING: GPUAcceleration disabled
```
我的系统环境为：
```
Ubuntu 22.04.1 LTS
NVIDIA Corporation GA100GL [A30 PCIe]
NVIDIA-SMI 525.78.01    Driver Version: 525.78.01    CUDA Version: 12.0 
$nvcc -V
Cuda compilation tools, release 12.0, V12.0.76
Build cuda_12.0.r12.0/compiler.31968024_0
```
环境变量设置：
```
$export |grep ABA
declare -x ABA_ACCELERATOR_TYPE="PLATFORM_CUDA"
```
`deviceQuery` 测试
```
deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 12.0, CUDA Runtime Version = 12.0, NumDevs = 1, Device0 = NVIDIA A30
Result = PASS
```

# 简单阐述原因：
**Abaqus2022 有硬缺陷，自带了libcuda之类的 低等级包，导致系统的cuda无法加载**

# 解决方案：
把Abaqus 自带的 libcuda 包 给规避或者删除掉，问题就能解决。
具体解决方案（如果是默认安装地址）：
移动以下文件：`libcuda.so`、`libcuda.so.1`和`libcuda.so.418.39`到新创建的 `keepcuda` 子目录，以便这些文件不会干扰系统上安装的驱动程序

1. 进入abaqus 自带的lib库
```
$cd /usr/SIMULIA/EstProducts/2022/EstPrd/linux_a64/code/bin
```
2. 创建规避文件夹：
```
$sudo mkdir keepcuda
```
3. 规避自带的cuda库
```
$mv libcuda.so ./keepcuda/libcuda.so
$mv libcuda.so.1 ./keepcuda/libcuda.so.1
$mv libcuda.so.418.39 ./keepcuda/libcuda.so.418.39
```

再运行 gpus=1 应该就不会出现 Error initializing the CUDA Driver NO_DEVICE 的问题。

***注：以上解决方案默认CUDA安装正确，且通过deviceQuery 测试，CUDA的权限没有问题。否则先确认系统环境是否设定ok**

# Workaround:
Create a subdirectory name keepcuda in the installation_dir/2022/EstPrd/linux_a64/code/bin directory. 
Move the following files: `libcuda.so`, `libcuda.so.1`, and `libcuda.so.418.39` 
to the newly created keepcuda subdirectory so that the files do not interfere with the driver installed on the system (in /usr/lib64). Note the subdirectory name is not important.