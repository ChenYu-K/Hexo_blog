---
title: How to run Abaqus under Linux and calling GPGPU for CUDA acceleration
date: 2023-02-03 15:01:24
tags: ABAQUS
toc: true
---

How to run Abaqus under Linux and calling GPGPU for CUDA acceleration

<!--more-->

**CUDA and Abaqus are installed correctly by default. And not the installed abaqus2022 (2022 has bugs that need to be fixed)**

*Note: You need to make sure CUDA is installed correctly under Linux, and the `deviceQuery` test shows that CUDA permissions are fine. Otherwise, first confirm that the installation environment is set ok, and then do the following.

If you don't change the defult address (note that the address is changed to your own abaqus version)
` /usr/SIMULIA/EstProducts/2022/linux_a64/SMA/site/abaqus_v6.env `
Modify `custom_v6.env` to work as well

The Linux setting is similar to the previous windows one, as follows.

1. add CUDA acceleration to the abaqus build environment
```
sudo vim /usr/SIMULIA/EstProducts/2022/linux_a64/SMA/site/abaqus_v6.env
```
2. Add the following command to the last line of `abaqus_v6.env`
```
os.environ["ABA_ACCELERATOR_TYPE"]="PLATFORM_CUDA" # Nvidia
```
3. add system environment variables
```
export ABA_ACCELERATOR_TYPE=PLATFORM_CUDA
```
After performing the above basic operations, you can perform CUDA acceleration if there are no other errors.
You need to add `gpus=1 ` at runtime