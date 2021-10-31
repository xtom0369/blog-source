---
title: 三大物理引擎之Bullet
date: 2021-10-28 22:38:04
tags:
- bullet physics
- physics
- unity
categories:
- physics
- unity
---

## <font color=#64EBC1>Why</font>

### 为什么要引入物理引擎？

常见的需求是<font color=#F46224>网络同步</font>。

当下通用的网络同步方案有<font color=#F46224>帧同步</font>和<font color=#F46224>状态同步</font>。状态同步自然是要在服务器跑物理，帧同步本来可以只跑客户端，服务器只负责转发命令（事实上我们上一个帧同步游戏也是这么做的），但是存在两个问题，一是不同平台存在不同步的可能，因为引擎的物理引擎存在浮点运算，不同平台可能运算结果不一致（这个问题我们使用了截断三位小数后面的方法，不同步出现的概率大大地降低了）；二是只跑客户端无法较好地做校验或者防作弊。

<!-- more -->

### 为什么是Bullet？

主要是项目接入的是Bullet，有一定的使用经验，另外Bullet的优点是项目结构相对简单，接入方便。当然Unity用的是[PhyX](https://developer.nvidia.com/physx-sdk)，后续有机会也会开PhyX的坑。

## <font color=#64EBC1>What</font>

### 学习Bullet的什么？

1. 通用形状的碰撞体实现

2. 碰撞检测实现

3. 刚体实现

4. 柔体实现

5. IK实现

6. 优化手段

    ......


## <font color=#64EBC1>How</font>

### 如何进行Bullet学习？

1. 文档和资料

  - [快速使用文档](https://github.com/bulletphysics/bullet3/blob/master/docs/BulletQuickstart.pdf)

    - {% post_link Bullet文档之《Bullet-User-Manual》阅读笔记 《Bullet-User-Manual》阅读笔记 %}


  - [用户手册](https://github.com/bulletphysics/bullet3/blob/master/docs/Bullet_User_Manual.pdf) 

2. Demo
   
  - [BulletPhysicsForUnity](https://assetstore.unity.com/packages/tools/physics/bullet-physics-for-unity-62991?locale=zh-CN)

    这是一个Unity版的Bullet插件，有[github](https://github.com/Phong13/BulletSharpUnity3d)版本，插件里将大量的C++接口暴露到了C#层，所以可以学到很多Bullet的概念，以及基础功能的实现。另外里面的bullet的dll是使用另一个github工程[BulletSharpPInvoke](https://github.com/Phong13/BulletSharpPInvoke)编译出来的，更底层的代码还是得看编译工程。

    - {% post_link BulletPhysicsForUnity下载和安装 下载和安装 %}


  - [bullet3](https://github.com/bulletphysics/bullet3)

    bullet的github仓库，根据教程编译出vs工程，设置启动项目，运行VS即可看到demo工程。

    ![](Demo.png)

3. 源码

  - [bullet3](https://github.com/bulletphysics/bullet3)

    撸完1和2，对bullet的使用也大概有个底了，就可以直接撸源码了。


未完待续 ...
    

