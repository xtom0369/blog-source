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

AStarPathfinding是最热门的寻路插件（下面简称<font color=#F46224>AStarPath</font>），对标的是Unity的[Navigation](https://docs.unity.cn/cn/2019.4/Manual/Navigation.html)，但是相比之下更加灵活，不仅支持多种不同的图，还能对图进行局部调整。

## <font color=#64EBC1>正文</font>

### 下载和安装

AStarPath可直接到[官网下载](https://arongranberg.com/astar/download)

![下载版本](下载版本.png)

官网分三个版本，免费版、Pro版和Beta版，可以在这里对比[不同版本之间的差异](https://arongranberg.com/astar/freevspro)，一个主要的差异还是免费版用不了Recast Graph，这个比较蛋疼，Recast相当于Navigation的自动生成路径，下面的文章基于4.2.15的Pro版本。

安装就直接创建一个Unity工程把插件的unitypackage导入即可。

### 基础概念

- Graph

AStarPath的核心概念是Graph，Graph可以理解为一张自定义生成导航数据的设置，注意是设置不是数据，生成的烘焙数据实际上在另一个地方。按照[官方的说法](https://arongranberg.com/astar/docs/getstarted.html)一个场景中最多可以有256张图，为了保证他足够的简单以及较好的性能，一般只用1~2个Graph来实现功能。

### 基本用法

为了防止枯燥还是直接上官方例子，主要是围绕插件支持的五种Graph，分别取一个例子。

- Recast Graph

场景ExampleScenes\Example3_Recast_Navmesh1中是Recast Graph的简单用法。

![RecastGraph](RecastGraph.png)

其中想要生成导航路径最核心的是挂在A*节点上的Pathfinder组件，对应的脚本为AstarPath，添加组件的菜单为**Component -> pathfinding -> pathfinder**。

![RecastGraph](AddNewGraph.png)

点击pathfinder上的AddNewGraph会弹出可选择的Graph，选择Recast Graph。

![Scan](Scan.png)

随后哪怕不修改默认设置，直接点击Scan按钮，就会生成一张默认的导航图（蓝色部分）。

- Grid Graph

- Layered Grid Graph

- Point Graph

- NavMesh Graph

