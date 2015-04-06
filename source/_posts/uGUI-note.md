title: uGUI_note
date: 2015-04-07 07:20:21
categories:
tags:
---
看Unity3D文档像看国内教课书一样，一些概念，不懂的时候看还是不懂，明白了以后再看，好像也没有说错。好几个做Unity3D的朋友跟我吐槽过U3D的文档质量，相比Apple贴心的技术文档相去甚远。

话虽这么说，权威的资料当然还是官方的，建议先仔细读一遍，不懂的再往下读：
[http://docs.unity3d.com/Manual/UISystem.html](http://docs.unity3d.com/Manual/UISystem.html)

网上的教程大多泛泛而谈，就不吐槽了。好在还找到了一篇比较好的解读，在此谢谢作者白猫。文章链接在下面
[http://www.cnblogs.com/whitecat/p/4159815.html](http://www.cnblogs.com/whitecat/p/4159815.html)

uGUI里面AutoLayout比较特别，我就此详细描述一下，希望能减少“教科书”的理解障碍。

#RectTransform的理解
UI元素有专门的RectTransform组件来描述元素的几何信息，继承于Transform，Inspector中其属性如下图。

![RectTransform属性](http://upload-images.jianshu.io/upload_images/301341-22cced0431fed13f.png)

##Anchors，锚点
我们首先需要认识Anchors。官网上的图片很好表达了Andhors的功能，请恕我实践下拿来主义。

![锚点全在中间的情况](http://upload-images.jianshu.io/upload_images/301341-220e11e70a5def02.gif)


![锚点全在右下角的情况](http://upload-images.jianshu.io/upload_images/301341-0c15d344dc0dad1c.gif)


![锚点在两个角的情况](http://upload-images.jianshu.io/upload_images/301341-c2f23bcd628ef49d.gif)


![锚点分开的情况](http://upload-images.jianshu.io/upload_images/301341-671094a181f26840.gif)

善于观察的同学可能已经发现，不管什么情况，button的四个顶点到相应锚点的相对位置是不变的：

![规律：button的四个顶点到相应锚点的相对位置是不变的](http://upload-images.jianshu.io/upload_images/301341-daaf44f85a3e6000.png)

当4个锚点都在一起的时候，RectTransform会显示Pos X，Pos Y，Width，Height四个属性可供修改。
![当4个锚点都在一起](http://upload-images.jianshu.io/upload_images/301341-dbe0f01f6d838000.png)

当不在一起的时候，RectTransform可调整的属性会有变化。

![锚点不在一起的时候，Inspector中变化1](http://upload-images.jianshu.io/upload_images/301341-198c0e7bbc414831.png)

![锚点不在一起的时候，Inspector中变化2](http://upload-images.jianshu.io/upload_images/301341-860dfe5e10e57a3b.png)


![锚点不在一起的时候，Inspector中变化3](http://upload-images.jianshu.io/upload_images/301341-7f477aa3a395b010.png)

Anchors的Min和Max分别是正规化的值(从0到1)，表示占父RectTransform的百分比，下图中AnchorMin=(0.1,0.1)  AnchorMax=(0.9,0.9)

![AnchorMin、AnchorMax](http://upload-images.jianshu.io/upload_images/301341-9f65e0d13aecc3be.png)

##Pivot，UI的中心点
UI元素的旋转和缩放是围绕pivot进行的。RectTransform组件中，Pivot属性是一个正规化的二维向量，用来描述中心点在本身矩形大小的位置。默认值为(0.5, 0.5)，即几何中心。

以上是RectTransform可视化的编辑属性，实际上RectTransform类的属性是怎样的？查阅RectTransform的官方文档如下：

[anchoredPosition]()
The position of the pivot of this RectTransform relative to the anchor reference point.

[anchoredPosition3D]()
The 3D position of the pivot of this RectTransform relative to the anchor reference point.

[anchorMax]()
The normalized position in the parent RectTransform that the upper right corner is anchored to.

[anchorMin]()
The normalized position in the parent RectTransform that the lower left corner is anchored to.

[offsetMax]()
The offset of the upper right corner of the rectangle relative to the upper right anchor.

[offsetMin]()
The offset of the lower left corner of the rectangle relative to the lower left anchor.

[pivot]()
The normalized position in this RectTransform that it rotates around.

[rect]()
The calculated rectangle in the local space of the Transform.

[sizeDelta]()
The size of this RectTransform relative to the distances between the anchors.

乍一看有点费解，这里有必要解释下：
####anchoredPosition
可以理解为Pivot点相对于Anchor reference点的位置。Anchor reference点，我是这样理解的：当四个anchors点在一起的时候，这个点就是anchor reference点；否则这四个点的中心点就是anchor reference点。
照这个推理，anchorMax和anchorMin的值将影响anchoredPosition的值。具体还需要demo研究来确认。[Bugfix]

但可以确定的是，当一个UI元素不需要自动拉伸行为时，用anchoredPosition + sizeDelta来设置位置和大小是比较方便的方法。

anchorMax、anchorMin和Inspector中的意义一致，需要注意的是，当UI元素不需要自动拉伸时，AnchorMax和AnchorMin是相等的。
![AnchorMin与AnchorMax](http://upload-images.jianshu.io/upload_images/301341-3f848986df9d0568.png)

UI元素需要自动拉伸时，使用anchorMax、anchorMin + offsetMax、offsetMin来设置UI的位置及大小会比较方便。
其中，anchorMax.x == anchorMin.x，height会自动拉伸，width固定不变。
anchorMax.y == anchorMin.y，width会自动拉伸，height固定不变。

不知不觉已过凌晨1点，先介绍到这里。关于uGUI的更多内容，将在下一篇博文中继续，敬请期待。
