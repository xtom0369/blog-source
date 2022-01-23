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

## <font color=#64EBC1>核心功能</font>

这是官方的宣传卖点

![优点1](优点1.png)

![优点2](优点2.png)

以下说下用下来之后感受比较深的点

- **分帧加载**

为什么把分帧放第一位，因为他确实给了我很大的启发，核心代码我在[分块加载](https://zhuanlan.zhihu.com/p/458672730)的文章里有贴，非常简单的几行代码，把分帧加载的思路展现得淋漓尽致。这种分帧的思路不仅可以用在资源层的加载，业务层同样可以使用，是一种特别有效的优化手段。

资源的加载流程一般包括**下载（针对边玩边下） -> 加载 -> 实例化 -> 加载回调**，这其中加载和加载回调是最不可控的，特别是加载回调，是接口调用者的代码，如果把这些流程放在一帧去执行就容易产生尖峰。

阅读源码可知，满载判断的API为**Updater.busy**，阈值为10ms，在

1. 下载（Download.UpdateAll）
2. 加载（Loadable.UpdateLoadingAndUnused）
3. 卸载（Loadable.UpdateLoadingAndUnused）
4. 实例化（InstantiateObject.Update）

都做了分帧处理。

**当然当前的分帧方案我觉得还有改进空间。**

1. 抛开单个资源过大的情况，分帧后仍有可能造成尖峰。

资源层用Updater.Update()来做总更新驱动，采用了多个管理器分别管理多种行为的设计，比如

1）下载管理器（Download）管理所有下载
2）加载器（Loadable，其实也是资源基类）管理所有的加载和卸载行为
3）实例化器（InstantiateObject）管理所有的实例化操作

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

其实我更偏向于把单个资源加载的所有流程当做统筹，下载（TODO，待测试） -> 加载 -> 实例化 -> 加载回调，单个步骤判断到满载就跳出，下一个循环从上次退出阶段往下走，从而降低三个阶段同时达到峰值的风险。

```CSharp
Queue<T> _reqQueue = new Queue<T>();
float _lastUpdateTime;

/// <summary>
/// 是否满载
/// </summary>
bool busy => Time.realtimeSinceStartup - _lastUpdateTime >= 0.005f;

/// <summary>
/// 每帧更新
/// </summary>
void Update()
{
    _lastUpdateTime = Time.realtimeSinceStartup;

    while (_reqQueue.Count > 0) 
    {
        var req = _reqQueue.Dequeue();
        
        // 加载和实例化

        if (busy) // 超过实例化的时间则分帧处理
            break;
    }
}

```

- **资源加载封装**

之前我们项目中这块其实没有特地封装过，所以导致工程后续出现了很多if代码，用于区分AB和非AB模式，后面加上了状态同步要共用部分客户端的代码，好家伙，大家一顿ctl+c ctl+v直接起飞。为了兼容AB和非AB两种模式，xasset在构建ab时同时生成了一个Manifest，用于记录每个bundle的信息，运行时利用Manifest生成asset到bundle的关系。

```CSharp
/// <summary>
/// 单个bundle的信息
/// </summary>
[Serializable]
public class ManifestBundle
{
    public string name;
    public List<string> assets;
    ......
}

/// <summary>
/// 每个build的中生成的所有bundle的信息
/// </summary>
public class Manifest : ScriptableObject
{
    public List<ManifestBundle> bundles = new List<ManifestBundle>();
    private Dictionary<string, ManifestBundle> nameWithBundles = new Dictionary<string, ManifestBundle>();
    
    /// <summary>
    /// 加载时构建asset到bundle的引用
    /// </summary>
    public void Load(string path)
    {
        nameWithBundles.Clear();
        foreach (var bundle in bundles)
        {
            nameWithBundles[bundle.name] = bundle;
            foreach (var asset in bundle.assets)
                nameWithBundles[asset] = bundle;
        }    }}
```

这样做有两个非常大的好处

1. 方便代码统一，不需要区分AB与非AB的方式来加载资源
2. 开发者加载时不需要关心资源从属于哪个bundle中。**注意，是加载时不需要关心，构建AB时是需要关心的。** 比如同个场景中的资源如果分散在太多ab中会增加场景加载时的IO。这个优点对后续优化资源大小或者优化AB构建规则提高加载速度有很大的帮助。

**但是，这种设计不兼容一种情况，就是下面这种直接拿bundle去取出所有的资源的时候**，当然官方也提供了接口去获取bundle（Versions.GetBundle），就是需要区分代码。


- **分布式增量打包**

[分布式增量打包](https://www.xasset.pro/docs/nocentral)也是xasset的卖点之一，这里需要先了解两个个概念，**Build和Group**，Build是构建分组，Group是单个构建规则，Build包含了一个或多个Group。

![Build和Group](Build和Group.png)

如图，我们定义了一个bytes的Build，是一个专门用来放二进制的分组，其中包含了配置表二进制config group和状态机二进制fsm group，当我们想更新了配置表或者状态机的二进制时，可点击底下的【Build Bundles】菜单进行单独构建，而不用对整个工程进行增量构建。

这是他的优点，也确实实现了这样的功能，下面我继续“鸡蛋里挑骨头”，说下这个设计当前存在的问题。

1. 不同Build之间最好不要存在资源依赖
2. 配置设计不完善，包括Target，Bundle Mode，Black List


- **资源加密**


热更新是否存在需要更新一大个包的问题



## <font color=#64EBC1>其他功能</font>

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

## <font color=#64EBC1>其他缺陷</font>




## <font color=#64EBC1>船新版本</font>


