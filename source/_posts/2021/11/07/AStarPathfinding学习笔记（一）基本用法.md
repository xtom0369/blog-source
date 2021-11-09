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

<!-- more -->

### 基础概念

- Graph

AStarPath的核心概念是Graph，Graph可以理解为一张自定义生成导航数据的设置，注意是设置不是数据，生成的烘焙数据实际上在另一个地方。按照[官方的说法](https://arongranberg.com/astar/docs/getstarted.html)一个场景中最多可以有256张图，为了保证他足够的简单以及较好的性能，一般只用1~2个Graph来实现功能。

### 基本用法

为了防止枯燥还是直接上官方例子，主要是围绕插件支持的五种Graph，分别取一个例子。

- Recast Graph

场景ExampleScenes\Example3_Recast_Navmesh1中是Recast Graph的简单用法。

![RecastGraph](RecastGraph.png)

1. 其中想要生成导航路径最核心的是挂在A*节点上的Pathfinder组件，对应的脚本为AstarPath.cs，添加组件的菜单为**Component -> pathfinding -> pathfinder**。

![RecastGraph](AddNewGraph.png)

2. 点击pathfinder上的**AddNewGraph**会弹出可选择的Graph，选择Recast Graph。

至此，生成导航数据的最基本步骤已经完成，用的是默认配置。

接下来的问题是怎么看到我们生成的导航图效果？直接点击Scene或者Inspector中的Scan按钮，就会生成一张默认的导航图（蓝色部分）。

![Scan](Scan.png)

然后Scan后的数据放在哪里？实际上此时的数据只在内存中，想要保存成文件有插件提供了两种方式，**Save和Cache**。

![Save](Save.png)

其生成的bytes文件一致，只不过Save偏向于**将数据保存至文件**，并且可自定义存储路径，Cache不仅会保存到文件，还会**设置为缓存文件**，使得AStarPath组件启动时不用重新执行Scan方法进行扫描，但是不能自定义存储路径。<font color=#F46224>商用的话不用想都是需要缓存的，我们试过地图较大时Scan的时间可能会长达10s+。</font> 

AStarPath.cs中可以看到这样一段判断代码

```CSharp
protected override void Awake () {
	base.Awake();
	......

	if (scanOnStartup && (!data.cacheStartup || data.file_cachedStartup == null)) {
		Scan();
	}
}
```

可以得知想要不触发运行时扫描只需要关闭**scanOnStartup**或打开**cacheStartup**缓存开关或设置缓存文件，对应着下图的三个选项。<font color=#F46224>需要注意的是，如果选择的是保存文件，需要另外把二进制文件设置到图中的③处缓存才会生效</font>

![ScanOnStartup](ScanOnStartup.png)


- Grid Graph

- Layered Grid Graph

- Point Graph

- NavMesh Graph

