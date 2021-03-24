---
title: VS2019代码提示延迟很高，部分快捷键无法使用的问题
date: 2021-02-26
tags: 
   - debug
   - Nvidia CUDA
   - Visual Studio
categories: 总结
copyright: true
---

### 原因 ###

- Visual Studio 与Nvidia CUDA NSightVSE冲突

### 现象 ###

- Tab、Ctrl'+C 、 Ctrl'+K 等快捷键失效，智能代码提示不能正常显示，有时不会出现，有时要等四五秒，非常不正常

> Setting中检查设置无误，尝试过重启、重置个人配置、重装等均无效
> 很长一段时间没有使用过vs，之前装了比较新的Nvidia CUDA，同时附带的有NSightVSE，也就是设计vs的拓展，问题出在这里
<!--more-->

### 解决方法 ###

> 卸载NSightVSE或者其他Nvidia和vs相关的拓展，重启vs后生效
> 正常情况搜索NsightVSE会出现四个，保留下列三个即可，版本仅供参考，可能其他版本不会出现如上问题。
> Nvidia CUDA Nsight NVTX 11.1
> Nvidia Nsight Compute 2020.2.0
> Nvidia Nsight Systems 2020.3.4
