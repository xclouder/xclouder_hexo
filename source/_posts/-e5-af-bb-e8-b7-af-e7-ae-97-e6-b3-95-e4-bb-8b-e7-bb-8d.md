title: 寻路算法介绍
id: 142
categories:
  - Unity3D
date: 2015-01-13 19:39:34
tags:
---

最近开展的新项目决定加入自动寻路的特性，以前一直觉得寻路是一个很神秘的东西，这次终于有机会揭开神秘的面纱，高兴。

网络上搜索寻路算法，首先映入眼帘的便是满屏的A*算法，可见这方面的资料已经非常多了。以下我就从项目实际需求出发，综合整理下这方面的资料，给大家节省点搜索的时间。

算法原理

关于A*算法的介绍，Patrick Lester有篇很好的文章，深入浅出：
[http://www.policyalmanac.org/games/aStarTutorial.htm](http://www.policyalmanac.org/games/aStarTutorial.htm) 

另外国人也有不错的翻译：
[http://blog.csdn.net/crayondeng/article/details/12342989](http://blog.csdn.net/crayondeng/article/details/12342989)
[http://www.cnblogs.com/technology/archive/2011/05/26/2058842.html](http://www.cnblogs.com/technology/archive/2011/05/26/2058842.html)

这是简单的A*算法，为了验证下效果，我们将游戏世界划分为网格，在这些网格上使用上面的A*算法，其结果如下图：

![astar_simple.png](http://xclouder-wordpress.stor.sinaapp.com/uploads/2015/01/astar_simple.png "astar_simple.png")

图中绿色线为路径，这走位很风骚，但显然不是我们想要的结果：两点之间线段最短嘛。

为了得到一个看起来合理的路径，要对上一步的到得结果进行处理，这个过程叫做路径平滑。

路径平滑

关于路径平滑算法也有不少文章进行了描述，不过google“路径平滑算法”结果较少，而“path smoothing algorithm”则能得到不少有价值的结果。

不少unity寻路插件都使用到了烟囱平滑算法，这篇文章图文并茂的介绍了算法的过程：

[http://digestingduck.blogspot.hk/2010/03/simple-stupid-funnel-algorithm.html](http://digestingduck.blogspot.hk/2010/03/simple-stupid-funnel-algorithm.html)

使用路径平滑后的寻路结果如下：

![NewImage](http://xclouder-wordpress.stor.sinaapp.com/uploads/2015/01/funnel_modify.png "funnel_modify.png")

如果要实现更加真实的路径平滑，还有很多因素需要考虑，比如当前速度、加速度、方向、最大转弯角度等等，这篇文章介绍了怎样获得更真实的路径结果：

[http://www.gamasutra.com/view/feature/3096/toward_more_realistic_pathfinding.php](http://www.gamasutra.com/view/feature/3096/toward_more_realistic_pathfinding.php)

部分寻路

在游戏中，用户指定的目的地有可能是一个不可达的区域，这时传统A*算法是找不到路径的。可是用户操作了什么都不做会很奇怪，比较好的做法是寻路到离用户指定地点最近的可达点，如下图：

![Door2 unity hero iPhone iPod Touch and iPad](http://xclouder-wordpress.stor.sinaapp.com/uploads/2015/01/Door2_unity_-_hero_-_iPhone__iPod_Touch_and_iPad.png "Door2_unity_-_hero_-_iPhone__iPod_Touch_and_iPad.png")

这个问题倒是挺好解决，因为A*算法是有启发式搜索特性的，当遍历完所有可搜索的节点后，如果还没有找到完整可达的路径，则循环结束时得到的路径（部分路径）就是我们想要的。这个结果取决于启发式分数算法，即[这篇文章](http://www.cnblogs.com/technology/archive/2011/05/26/2058842.html)中讲到的H值计算方法。

动态障碍

动态障碍也是游戏中经常会遇到的情况，比如迷宫关着的门，有钥匙可以打开；再比如怪物挡住去路，要杀掉才能通过等。动态障碍会动态地改变地图节点的可通过信息，需要实现的很高效。关键点在批量刷新，并且只刷新有变化的区域。具体细节，我在后续的文章中会有介绍。

这些是最基本的寻路相关问题，针对不同场景，寻路算法也会有相应的优化，以下是我找到的一些资料，有兴趣可以看看：

http://www.zhihu.com/question/20298134（知乎，你懂的）

http://blianchen.blog.163.com/blog/static/13105629920103614613291/ （几何知识科普）

http://blianchen.blog.163.com/blog/static/1310562992010324046930/（导航网格寻路）

http://blianchen.blog.163.com/blog/static/13105629920103211052958/ （路点寻路）

http://aigamedev.com/open/tutorials/potential-fields/ （势能寻路）