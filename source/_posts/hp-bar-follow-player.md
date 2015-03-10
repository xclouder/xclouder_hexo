title: 让血条跟随角色
date: 2015-03-08 23:17:48
categories: Unity3D
tags: hpbar
---
目前实现方案采用的是： 获取人物世界坐标 -> 转换到UI层坐标 -> 创建血条UI对象并指定位置显示。

但是调试发现血条会抖动，看起来血条位置刷新不一致。

检查脚本逻辑，发现相机跟随角色逻辑和血条刷新逻辑都在LateUpdate方法里，但是这里的执行顺序不明朗。
如果要获取正确的结果，逻辑和执行顺序应该是这样：  
1. Player移动  
2. Camera跟随移动  
3. UI血条位置更新  

那么就是说脚本的Update、LateUpdate执行顺序应该是这样：  
1. Player移动脚本  
2. CameraFollow脚本  
3. HpDisplay脚本  

脚本默认按加载顺序来决定Update执行顺序，我们可以到Edit > Project Settings > Script Execution Order里自定义顺序。如下图：  
![](img/4.png)

图中"Default Time"是默认脚本的执行顺序，没有特别指定顺序的脚本，都在"Default Time"期间执行，他们之间的执行顺序由脚本加载时机决定。

"Default Time"以上的脚本，执行更靠前；"Default Time"以后的脚本，在默认脚本之后执行。

按上述机制重新调整脚本逻辑后，血条抖动问题得以解决。