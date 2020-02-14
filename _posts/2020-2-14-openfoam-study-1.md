---
date: 2019-05-16T23:48:05.000Z
layout: post
title: OpenFOAM初探（一）
subtitle: 基本环境配置
description: >-
    本文主要探讨OpenFOAM的运行环境
image: ../article_img/2020-2-14-openfoam-study-1/title.jpg
optimized_image: ../article_img/2020-2-14-openfoam-study-1/title.jpg
category: CFD
tags:
  - CFD
  - OpenFOAM
  - Linux
author: Xiaoxiao
paginate: true
---

## 前言

<p style="text-indent:2em">笔者早年的工作主要在win10平台进行，使用过诸如CFX、Fluent等商业软件，后续论文工作主要操练的是Fluent UDF。诚然，Fluent依托UDF在可操作性上已经远超其余商业CFD软件(待考证)，但是其缺陷依然明显：1, 界面操作繁琐，对于一些参数化的研究工作，很难通过脚本实现(GUI依赖版本，TUI懒得学)；2, 新功能的整合程度参差不齐，以笔者常用的DPM为例，功能限制较多，BUG不少...；3, 很难修改顶层求解器，很多中间步骤需要手工；4, 安装包太大，正版太贵。</p>

<p style="text-indent:2em">OpenFOAM是一款流行的开源CFD工具箱，最初萌生体验的想法主要还是觉得自己的代码水平有些上路了(自我感觉)，此外觉得用命令行和脚本操作很有一种Geek的感觉～</p>

## 环境配置
<p style="text-indent:2em">参考官网(https://openfoam.org/)，OpenFOAM主要面向Linux，同时也一定程度支持Windows 10和macOS。笔者在研究过程中大概换了以下这些环境:</p>
### Windows 10 WSL
<p style="text-indent:2em">虽然按照官网的说法，</p>
> OpenFOAM is packaged for simple installation on Ubuntu Linux, **which can be directly installed on Windows 10** and is available as a Docker image for other Linux and macOS.

<p style="text-indent:2em">但是事实上这是指WSL，即Windows Subsystem Linux，具体的安装和操作指南可见[了解适用于 Linux 的 Windows 子系统](https://docs.microsoft.com/zh-cn/windows/wsl/about)。</p>
WSL的易用性确实非常优秀，在上述的微软官网下载Ubuntu 18.04，简单配置一下，再按照OpenFOAM的官方安装指南就完事了。缺点如下：1，性能较原生Linux要低，尤其是I/O性能相当糟糕(WSL2有所改进)，不少算例耗费的时间要多出30%～50%不等；2，可视化依赖第三方，不支持高版本OpenGL，这就意味着使用者不能直接使用paraFoam做后处理。如果要呈现拉格朗日粒子，这是个致命的缺陷；3，Windows和UNIX系统的文件命名规则不同，转换之间会出现非法字符（比如冒号）。</p>
<p style="text-indent:2em">当然，如果是在Windows上办公，后台挂个小算例这种场景，WSL是几乎完美的。双系统用户对硬盘空间不敏感的话完全可以装一个。</p>
### Deepin 15.11
<p style="text-indent:2em">国产的Linux系统，优点是本地化到位，自带QQ、微信、百度盘这种应用，符合国人操作习惯。OpenFOAM安装起来和WSL、Ubuntu一模一样。缺点是桌面性能不好，尤其是显卡驱动支持稀烂，对双显卡的笔记本支持很差，界面虽然很漂亮但是很多组件都卡卡的，很多窗口甚至会出现一些图形的错位，自带的记事本更是卡的要命...用了两天果断放弃。</p>
### Manjaro Gnome
<p style="text-indent:2em">基于Arch的Linux系统，在科学计算领域相对于Ubuntu、CentOS/RedHat这些系列似乎要冷门不少。但是体验下来桌面确实很漂亮，性能也很棒！同时，Gnome支持HIDpi，完美适配4K屏幕，N卡的驱动(大黄蜂)默认已经自动打上了。商店软件很全，连CUDA、CuCNN这种包都能直接下...总体感觉pacman的体验要优于apt。</p>
<p style="text-indent:2em">虽然官网没有直接的指导，但前置包的安装其实比其他系统更加简洁：</p>

```bash
sudo pacman -S gcc qt flex cmake openmpi git
```

注：这里搜qt会出很多包，不知道哪些是必要的，心一横就全安了=。=
