title: AstarPathFinding Project的2D扩展
date: 2015-02-28 15:06:43
categories:
  - Unity3D
tags:
---

AstartPathFinding是Unity里一个非常高效的A*寻路插件，支持GridGraph、Navmeshes、PointGraph、动态障碍、可扩展等特性(特性列表：[http://arongranberg.com/astar/features](http://arongranberg.com/astar/features))。

可惜的是目前不支持2D，不过没关系，既然支持扩展，咱们可以自己实现一个2D扩展，顺便阅读源码学习学习(不得不赞一下作者，注释写的很全)。

<!--more-->

开始扩展之前，我们先通过阅读代码了解下这个插件的结构。
整个系统API由AstarPath类提供，寻路接口如下
```
public static void StartPath (Path p, bool pushToFront = false)
```
这个接口是异步的，调用这个方法开始寻路后，路径的计算会经过5个状态：创建、等待计算、正在计算、返回等待、已返回。从Path类的PathStatus枚举属性可以看出来：
```
public enum PathState {
    Created = 0,
    PathQueue = 1,
    Processing = 2,
    ReturnQueue = 3,
    Returned = 4
}
```
返回的路径会通过回调函数通知上层逻辑。

A*寻路核心逻辑在AstarPath类的两个CalculatePaths方法中：
```
private static IEnumerator CalculatePaths (System.Object _threadInfo)
void CalculatePathsThreaded (System.Object _threadInfo)
```

第一个是协同的版本，使用StartCorotine来调用，不使用额外的线程；第二个是多线程的版本，可以根据项目需要进行调优。

Path是一个抽象类，用来将不同寻路算法进行封装，具体子类有：ABPath、ConstantPath、FleePath、FloodPath、FloodPathTracer、MultiTargetPath、RandomPath。

Path计算过程中，会读取地图信息——NavGraph，最后得出的路径为GraphNode的列表。NavGraph是由扫描场景信息获取的。见AstarPath类中的Scan方法。

寻路的算法，咱们直接延用ABPath就好，剩下还差一个2D的NavGraph。插件原有一个GridGraph，本是3D的，经测试旋转一下也能作为2D使用，但需要经过一些设置。而且相应的动态障碍组件DynamicGridObstacle还不支持2D。索性我们仿照GridGraph专门写一个GridGraph2D来支持2D。

官方网站对扩展有个简单的文档，但似乎很久没更新了，新版的接口已经发生了改变。下面了记录我的实现过程：
###1.实现GridGraph2D
考虑到原有的GridGraph扩展起来并不方便，我这里让GridGraph2D直接继承自NavGraph，并实现IUpdatableGraph和IRaycastableGraph接口。

几个关键的方法需要重写
```
public override void ScanInternal (OnScanStatus statusCallback)
//这个方法扫描场景，将地图网格化，并记录每个格子是否可走(walkable)。

public override void GetNodes (GraphNodeDelegateCancelable del)
//获取地图的所有网格节点

void UpdateArea (GraphUpdateObject o)
//对地图节点可行走状态进行局部刷新，实现这个才能支持动态障碍。
```
可以参照原来GridGraph的实现，主要将3d中xz坐标，换到xy坐标

具体代码请参看附件。

###2.实现GraphNode2D
同上，直接继承GraphNode，并参考GridNode实现。

###3.实现GridGraph2DEditor
在AstarPath组件的Inspector中，有各种Graph的可视化参数编辑，我们可以通过实现一个GraphEditor来为上面写的GridGraph2D添加一个可视化编辑器。
![](/images/1.png)

在类的上面，添加如上图所示的Attribute，即可在AstarPath的Inspector中看到新建“Grid Graph 2D”的按钮：
![](/images/2.png)

效果如下：
![](/images/3.png)

###4.实现DynamicGridObstacle2D
在原来DynamicGridObstacle基础上，改变坐标、并去掉高度检测。

###5.实现FunnelModifier2D
FunnelModifier是烟囱路径平滑算法，AstarPathFinding中，路径平滑是作为一个处理链存在的，即获得A*算法的初步路径后，可以经过多个路径平滑算法处理，最终得到更理想的路径。原来的FunnelModifier也不支持2D，需要修改坐标xz->xy。
