title: NGUI Sprite拉伸边缘虚化问题
tags:
  - NGUI
  - Sprite
id: 153
categories:
  - Unity3D
date: 2015-01-30 17:58:07
---

项目里有个需求，需要在某个UI元素下面放置一个纯色的下划线，我使用的2x2的白色方块，然后进行拉伸实现。
但是问题来了，当我把这个方块加入图集中后，创建的UISprite看上去是这个样子：
![](/images/520e20bf15de9c63690fc58e5d675f24e5e79730.png)

说好的纯色呢？

Google一番未果，于是跑到NGUI官方论坛上问了一下，第二天就有了回复。原来我的UISprite是在图集中，而生成图集的时候，默认使用的Filter Mode 是Trilinear，这种线性插值导致渲染时会和周围的像素“渗透”。如果把图集的Filter Mode设置成Point，就没有这个问题了。

![](/images/644040de72e2e11ca33f18fa82663628147d5b74.jpg)


![319285CF 64CD 4211 BFC3 DFDBF3A9D522](/images/663c4fcf52b8925b2dd2108db9cba0079d7c400e.png)

但是设置成Point会强制要求同一个图集里的UISprite都使用这个模式，可能会影响效果。
一个折衷的方案是，把这类素材放在单独的图集里，但这样会增加DrawCall。
更好的方法是Sprite仍在原来图集中，使用Sprite的Sliced模式，
![](/images/44021803f3589cdc1cf8cef011b15e431e8f0e04.png)
并给Sprite设置一个1px的边框
![](/images/836c626831d594bfd02700c72fb401c6368fe6f6.png)
这样图片能正常显示和拉伸了。这种方案性能会更好。

###【Tips】
关于FilterMode，它的意义是：
当该纹理由于3D变换进行拉伸时，它将如何被过滤插值。共有三种选择：
| FilterMode | 意义 |
|:----------:| ---- |
| Point | 单点插值，纹理将变得块状化（blocky up close）|
| Bilinear | 双线性插值，纹理将变得模糊（blurry up close）|
| Trilinear | 三线性插值，类似Bilinear，但是纹理还会在不同的mip水平之间（between the different mip levels）进行模糊 |

###【参考】
[http://www.tasharen.com/forum/index.php?topic=12439.msg56762#msg56762](http://www.tasharen.com/forum/index.php?topic=12439.msg56762#msg56762)
[http://docs.unity3d.com/ScriptReference/FilterMode.html](http://docs.unity3d.com/ScriptReference/FilterMode.html)
[http://en.wikipedia.org/wiki/Texture_filtering](http://en.wikipedia.org/wiki/Texture_filtering)