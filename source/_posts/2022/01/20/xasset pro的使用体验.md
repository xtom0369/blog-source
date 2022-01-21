---
title: xasset pro的使用体验
date: 2022-01-20 21:36:27
tags:
- Unity
- 资源加载
- 分帧加载
- xasset
categories:
- Unity
- 资源加载
---

最近把项目的资源层换成了[xasset团队版](https://www.xasset.pro/)，有收获，也有低于预期的地方，下面我会把优点及不足的地方一一列举，但总的来说瑕不掩瑜，作为一个3000块的产品，还是值得的，产品确实有打磨过的痕迹，代码再多次精简之后，阅读性和可维护性较强。下面我重点写他的使用体验，操作层面直接看文档或者看源码即可。好像在我们买完不久作者就给免费版开放了非常多功能。

## <font color=#64EBC1>目的</font>

上一篇[大地图分块加载](https://zhuanlan.zhihu.com/p/458672730)的文章里有提到，当时我们有了一个**流畅分帧加载**的需求，我们之前的项目主要是分量加载，比如最多同时加载10的资源，这种设计的问题我在上一篇文章也提到了，加载10个10M的资源和10个10k的资源是不一样的事情，没办法实现平滑加载，而分帧加载是xasset pro的宣传点之一。

其次是边玩边下的功能，这个功能只是用demo简单玩了下和粗略过了一下源码，还没深入使用，以后用了再回来补充吧。但是和专门做这块的快乐变化公司的边玩边下（用过），价格相比绝对是碾压，其次那种全套源码在手做扩展做定制化真的很爽，特别是代码可阅读性较高的时候。

所以综合考虑下购买了团队版服务。

## <font color=#64EBC1>优缺点</font>

这是官方的宣传卖点

![优点1](优点1.png)

![优点2](优点2.png)

以下说下用下来之后感受比较深的点

- **分帧加载**

为什么把分帧放第一位，因为他确实给了我很大的启发，核心代码我在[分块加载](https://zhuanlan.zhihu.com/p/458672730)的文章里有贴，非常简单的几行代码，把分帧加载的思路展现得淋漓尽致。这种分帧的思路不仅可以用在资源层的加载，业务层同样可以使用，是一种特别有效的优化手段。

资源的加载流程一般包括**下载 -> 加载 -> 实例化 -> 加载回调**，这其中加载和加载回调是最不可控的，特别是加载回调，是接口调用者的代码，如果把这些流程放在一帧去执行就容易产生尖峰。

阅读源码可知，满载判断的API为**Updater.busy**，阈值为10ms，在

1. 下载（Download.UpdateAll）
2. 加载（Loadable.UpdateLoadingAndUnused）
3. 卸载（Loadable.UpdateLoadingAndUnused）
4. 实例化（InstantiateObject.Update）

都做了分帧处理。

**当然当前的分帧方案我觉得还有改进空间。**

1. 抛开单个资源过大的情况，分帧后仍有可能造成尖峰。

资源层Updater.Update()来做总更新驱动，采用了多个管理器分别管理多种行为的设计，比如

1. 下载管理器（Download）管理所有下载
2. 加载器（Loadable）管理所有的加载和卸载行为
3. 实例化器（InstantiateObject）管理所有的实例化操作

Updater.Update代码如下
```CSharp
private void Update()
{
    realtimeSinceUpdateStartup = Time.realtimeSinceStartup;
    ...
    Loadable.UpdateLoadingAndUnused();
    Operation.UpdateAll(); // InstantiateObject是Operation的派生类
    Download.UpdateAll();
}
```
满载busy的判断分别在各个管理器的更新方法中，这样一来当**加载/实例化/下载**同时满载时，其实分帧加载的峰值是可以达到30ms+，且加载回调没有纳入分帧满载的判断中，也是一个不可控因素。

- **资源路径封装**

之前这块其实没有特地封装过，





## <font color=#64EBC1>缺点</font>

没有设计分帧加载的开关




```CSharp
public sealed class InstantiateObject : Operation
{
    private static readonly List<InstantiateObject> AllObjects = new List<InstantiateObject>();
    private Asset _asset;
    private string _path { get; set; }

    public override void Start()
    {
        base.Start();
        _asset = Asset.LoadAsync(_path, typeof(GameObject));
        AllObjects.Add(this);
    }

    public static InstantiateObject InstantiateAsync(string assetPath)
    {
        var operation = new InstantiateObject
        {
            _path = assetPath
        };
        operation.Start();
        return operation;
    }

    ....
}
```

有问题其实很明显，一是实例化的
