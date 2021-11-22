---
title: win11拖拽BUG解决方案
date: 2021-10-28 16:57:53
tags: system
---

# strange lag on dragging windows around
---

A somewhat niche occurrence that a handful of users anre seeing, is pointer gost effect <font color=red>when dragging some windows around</font>. <br>
the lag doesn't ook to be a bug with Windows 11 directly, as the author of one post notes "i noteced that the lag only started when the window overlapped with the task bar. This made me remember that i have open shell installed. Uninstalling this fixed the issue." <br>
It seems Windows 11 isn't a compbatible with some taskbar-chaing software you might be runging on windows 10 anyway, so just make sure to double check before installing.

## 修复方式
---

window11 拖拽延迟问题很有可能是因为文件夹管理软件得不兼容造成。 当关闭文件夹，或者让文件夹不和任何窗口重叠就不会发生类似问题。<br>
这里，我因为下载了`QTTabBar`，所以出现了拖拽延迟，卸载它，或者安装它最新支持win11的版本，可以解决问题。<br>

## 其他解决方案
--- 
类似的拖拽延迟，网络上解决方案有很多，汇总起来大概是：

- 调低鼠标回报率
- 关闭窗口透明
- 关闭性能显示效果
- 跟新显卡驱动
