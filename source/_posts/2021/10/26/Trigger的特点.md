---
title: Trigger的特点
date: 2021-10-26 23:23:44
tags:
- Unity
- Animator
- Trigger
categories:
- Unity
- Animator
---

## Trigger的特点

众所周知，Animator的Trigger和Bool的区别是，Trigger是一次性触发判断，触发后会自动重置，而Bool是持久性变量，所以使用Trigger图的就是自动重置的功能。

<!-- more -->

## 使用例子

举个简单的例子，这是一个宝箱的Animator

![简单例子](1.png)

例子中的Animator初始状态为**idle**，设置触发器<span style="background-color: #F9F2F4;"><font color=#CE3858>**Open**</font></span>则跳转到状态**open**，设置触发器<span style="background-color: #F9F2F4;"><font color=#CE3858>**Idle**</font></span>则跳转到状态**Idle**。

由于Trigger是**消耗型**参数，所以参数只有在使用后才会发生重置。所以以下的两种写法最后都会得到相同的结果。

- 写法一

```CSharp
animator.SetTrigger("Open");
animator.SetTrigger("Idle");
```

- 写法二

```CSharp
animator.SetTrigger("Idle");
animator.SetTrigger("Open");
```

原因是方法二设置<span style="background-color: #F9F2F4;"><font color=#CE3858>**Idle**</font></span>触发器后并没有发生消耗，等到切换到open状态后才开始判断，消耗<span style="background-color: #F9F2F4;"><font color=#CE3858>**Idle**</font></span>触发器，跳转到**idle**状态，得到和方法一一致的效果。

