---
title: How to use right-click to open CMD
toc: false
date: 2021-08-04 21:53:33
tags: Tips
categories: computer science
---
# 右クリックでCMDを素早く起動する方法</br>如何通过右键快速启动CMD

![](https://img.icons8.com/ios-glyphs/30/000000/cmd.png)
Most of time, when we want op the cmd. we should open CMD frst, and then copy the path ,use `cd 'path'` to start command.

Now,you can open CMD from the current path by modifying the registry.

<!--more-->

Creat a 'xx.reg', copy the following code into it,and then run it.
```bash
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\shell\OpenCmdHere]
@="open cmd"
"Icon"="cmd.exe"
[HKEY_CLASSES_ROOT\Directory\shell\OpeCmdHere\command]
@="cmd.exe /s /k pushd \"%V\""
[HKEY_CLASSES_ROOT\Directory\Background\shell\OpenCmdHere]
@="open cmd"
"Icon"="cmd.exe"
[HKEY_CLASSES_ROOT\Directory\Background\shell\OpenCmdHere\command]
@="cmd.exe /s /k pushd \"%V\""
[HKEY_CLASSES_ROOT\Drive\shell\OpenCmdHere]
@="open cmd"
"Icon"="cmd.exe"
[HKEY_CLASSES_ROOT\Drive\shell\OpenCmdHere\command]
@="cmd.exe /s /k pushd \"%V\""
[HKEY_CLASSES_ROOT\LibraryFolder\background\shell\OpenCmdHere]
@="open cmd"
"Icon"="cmd.exe"
[HKEY_CLASSES_ROOT\LibraryFolder\background\shell\OpenCmdHere\command]
@="cmd.exe /s /k pushd \"%V\""
```