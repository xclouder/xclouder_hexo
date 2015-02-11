title: NGUI Sprite拉伸边缘虚化问题
tags:
  - NGUI
  - Sprite
id: 153
categories:
  - Unity3D
date: 2015-01-30 17:58:07
---

<div>项目里有个需求，需要在某个UI元素下面放置一个纯色的下划线，我使用的2x2的白色方块，然后进行拉伸实现。</div>
<div>但是问题来了，当我把这个方块加入图集中后，创建的UISprite看上去是这个样子：</div>
<div> ![08331697 70BB 4638 AE4D AC543A3FF79A](http://xclouder-wordpress.stor.sinaapp.com/uploads/2015/01/08331697-70BB-4638-AE4D-AC543A3FF79A.png)</div>
<div>说好的纯色呢？</div>
<div> </div>
<div>Google一番未果，于是跑到NGUI官方论坛上问了一下，第二天就有了回复。原来我的UISprite是在图集中，而生成图集的时候，默认使用的Filter Mode 是Trilinear，这种线性插值导致渲染时会和周围的像素“渗透”。如果把图集的Filter Mode设置成Point，就没有这个问题了。 </div>

![282BD608 73DD 4788 8B9E 97BAE9DF865B](http://xclouder-wordpress.stor.sinaapp.com/uploads/2015/01/282BD608-73DD-4788-8B9E-97BAE9DF865B.jpg)


<div>![319285CF 64CD 4211 BFC3 DFDBF3A9D522](http://xclouder-wordpress.stor.sinaapp.com/uploads/2015/01/319285CF-64CD-4211-BFC3-DFDBF3A9D522.png)</div>
<div> </div>
<div>但是设置成Point会强制要求同一个图集里的UISprite都使用这个模式，可能会影响效果。</div>
<div>一个折衷的方案是，把这类素材放在单独的图集里，但这样会增加DrawCall。</div>
<div> </div>
<div>更好的方法是Sprite仍在原来图集中，使用Sprite的Sliced模式，</div>
<div>![Untitled TestNGUI PC Mac Linux Standalone](http://xclouder-wordpress.stor.sinaapp.com/uploads/2015/01/Untitled_-_TestNGUI_-_PC__Mac___Linux_Standalone1.png)</div>
<div> </div>
<div>并给Sprite设置一个1px的边框</div>
<div>![Untitled TestNGUI PC Mac Linux Standalone](http://xclouder-wordpress.stor.sinaapp.com/uploads/2015/01/Untitled_-_TestNGUI_-_PC__Mac___Linux_Standalone.png)</div>
<div>这样图片能正常显示和拉伸了。这种方案性能会更好。</div>
<div> </div>
<div>【Tips】</div>
<div>关于FilterMode，它的意义是：</div>
<div style="padding: 0px">
<div><span style="font-family: Arial;font-size: 14px">当该纹理由于3D变换进行拉伸时，它将如何被过滤插值。共有三种选择：</span></div>
</div>
<div><span style="font-family: Arial;font-size: 14px">Point</span> <span style="font-family: Arial;font-size: 14px">单点插值，纹理将变得块状化（</span><span style="font-family: Arial;font-size: 14px">blocky up close</span><span style="font-family: Arial;font-size: 14px">）；</span>
<div><span style="font-family: Arial;font-size: 14px">Bilinear</span> <span style="font-family: Arial;font-size: 14px">双线性插值，纹理将变得模糊（</span><span style="font-family: Arial;font-size: 14px">blurry up clos</span><span style="font-family: Arial;font-size: 14px">e</span><span style="font-family: Arial;font-size: 14px">）；</span></div>
<div><span style="font-family: Arial;font-size: 14px">Trilinear</span> <span style="font-family: Arial;font-size: 14px">三线性插值，类似</span><span style="font-family: Arial;font-size: 14px">Bilinear</span><span style="font-family: Arial;font-size: 14px">，但是纹理还会在不同的mip水平之间（</span><span style="font-family: Arial;font-size: 14px">between the different mip levels</span><span style="font-family: Arial;font-size: 14px">）进行模糊；</span></div>
</div>
<div> </div>
<div>【参考】</div>
<div>[http://www.tasharen.com/forum/index.php?topic=12439.msg56762#msg56762](http://www.tasharen.com/forum/index.php?topic=12439.msg56762#msg56762)</div>
<div>[http://docs.unity3d.com/ScriptReference/FilterMode.html](http://docs.unity3d.com/ScriptReference/FilterMode.html)</div>
<div>[http://en.wikipedia.org/wiki/Texture_filtering](http://en.wikipedia.org/wiki/Texture_filtering)</div>