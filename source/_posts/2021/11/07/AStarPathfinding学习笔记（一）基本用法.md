---
title: AStarPathfinding学习笔记（一）基本用法
date: 2021-11-07 18:21:15
tags:
- Unity
- Pathfinding
- A*
categories:
- Unity
- Pathfinding
---

## <font color=#64EBC1>前言</font>

AStarPathfinding是最热门的寻路插件，对标的是Unity的[Navigation](https://docs.unity.cn/cn/2019.4/Manual/Navigation.html)，但是相比之下更加灵活，不仅支持多种不同的图，还能对图进行局部调整。

## <font color=#64EBC1>正文</font>

### 下载和安装

AStarPathfinding可直接到[官网下载](https://arongranberg.com/astar/download)

![下载版本](下载版本.png)

由图可见官网分三个版本，免费版、Pro版和Beta版，可以在这里对比[不同版本之间的差异](https://arongranberg.com/astar/freevspro)。

![版本差别](版本差别.png)

一个主要的差异还是用不了Recast，这个比较蛋疼，Recast相当于Navigation的自动生成路劲，下面的文章基于Pro版本。

安装就直接创建一个Unity工程把插件的unitypackage导入即可。


