---
title: BulletPhysicsForUnity下载和安装
date: 2021-10-31 17:41:01
tags:
- BulletPhysicsForUnity
- bullet physics
- physics
- unity
categories:
- physics
- unity
---

## <font color=#64EBC1>前言</font>

### BulletPhysicsForUnity是什么？

BulletPhysicsForUnity是Bullet物理引擎在Unity上的插件。

<!-- more -->

### BulletPhysicsForUnity要下载哪个版本？

网上一共可以找到两个版本，分别是[assetstore版本](https://assetstore.unity.com/packages/tools/physics/bullet-physics-for-unity-62991?locale=zh-CN)和[github版本](https://github.com/Phong13/BulletSharpUnity3d)。两个版本其实差不多，github版本可以看到最新的仓库，当然还有最新的bug。这两个仓库的共同点是都很多年没维护了。

### BulletPhysicsForUnity和[BulletSharpPInvoke](https://github.com/Phong13/BulletSharpPInvoke)有什么关系？

BulletSharpPInvoke是BulletPhysicsForUnity中C++ 源码的编译库。也就是说BulletPhysicsForUnity中只包含了Bullet的编译库以及暴露到C#的接口，不包含Bullet的源代码，但是这并不妨碍入门。

### BulletPhysicsForUnity可以直接商用么？

不确定，但当前存在的问题是接口的封装和Unity原生的相比比较原始，另外这种插件一般都需要另外做性能优化，因为游戏里的样本量和demo不是一个数量级。

## <font color=#64EBC1>下载和安装</font>

### 下载

1. [assetstore仓库](https://assetstore.unity.com/packages/tools/physics/bullet-physics-for-unity-62991?locale=zh-CN)
2. [github仓库](https://github.com/Phong13/BulletSharpUnity3d)

### 安装

两个仓库都是创建一个Unity空工程然后选1的把资源Import进工程，选2的把仓库放到Assets目录下即可。

导入后选择Assets\Examples\Scenes\ExampleMenu场景，启动即可查看Demo效果。

![](1.png)

至此，BulletPhysicsForUnity就下载安装完成。
