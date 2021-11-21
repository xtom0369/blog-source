---
title: Unity的Collider参数需要注意的点
date: 2021-11-20 23:42:16
tags:
- Collider
- Physics
- Unity
- Bullet Physics
categories:
- Physics
- Unity
- Bullet Physics
---

## <font color=#64EBC1>前言</font>

问题的起因是项目中物理引擎的效果和运行时不一致，出现了一些虚空阻挡，或者碰撞效果异常，最后查到是对Collider组件中center等参数理解有偏差，<font color=#F46224>部分参数的配置需要叠加GameObject的rotation和scale的效果才是最终的结果</font>。

## <font color=#64EBC1>详解</font>

由于物理引擎用的是Bullet，以BoxCollider为例，

![BoxCollider](BoxCollider.png)

Bullet底层只认transform和boxHalfExtents（boxHalfExtents为size的一半），并没有center的概念，所以需要把center叠加到position上，以及把size换算为boxHalfExtents后传递给Bullet。

这里有个需要注意的点是，center和size都是给予BoxCollider所在的GameObject的局部设置，所以center和size的换算需要叠加GameObject的rotation以及scale，代码如下。

```CSharp
/// <summary>
/// transform.position叠加了center后的世界坐标
/// </summary>
public Vector3 worldPosition => transform.position + Vector3.Scale(transform.rotation * center, transform.lossyScale);

public Vector3 boxHalfExtents => Vector3.Scale(transform.rotation * size / 2, transform.lossyScale);
```