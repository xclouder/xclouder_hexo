title: Unity3d 编译log分析
id: 123
categories:
  - Unity3D
date: 2014-12-19 20:10:32
tags:
---

Unity编译过程中一些重要的信息输出在Editor.log中，mac下默认位置是~/Library/Logs/Unity/Editor.log

或者通过Console面板右上角下拉菜单-》Open Editor Log打开

里面有一些信息对分析包大小、查找无效脚本有帮助。

例如： Script attached to 'ShopContainer' in scene '' is missing or no valid script is attached. 一般是对象上挂载的脚本被删或无效导致

在优化包大小时，想检查dll引用，可以在log中找到如下信息

Mono dependencies included in the build 
Dependency assembly - Mono.Posix.dll 
Dependency assembly - Mono.Security.dll 
Dependency assembly - System.Configuration.dll
Dependency assembly - System.Core.dll 
Dependency assembly - System.Security.dll
Dependency assembly - System.Xml.dll
 Dependency assembly - System.dll 
Dependency assembly - mscorlib.dll 
Dependency assembly - UnityEngine.UI.dll 
Dependency assembly - IFacebook.dll 
Dependency assembly - P31RestKit.dll 
Dependency assembly - UniExtensions.dll 
Dependency assembly - bouncywp71.dll
 Dependency assembly - JDKLog.dll 
Dependency assembly - Assembly-CSharp.dll 
Dependency assembly - Assembly-UnityScript-firstpass.dll

Log中还列出了压缩前各素材用量占比，如：

Textures 224.1 mb 93.8% 
Meshes 0.0 kb 0.0% 
Animations 173.2 kb 0.1% 
Sounds 2.1 mb 0.9% 
Shaders 74.9 kb 0.0% 
Other Assets 2.8 mb 1.2% 
Levels 540.5 kb 0.2% 
Scripts 2.7 mb 1.1% 
Included DLLs 6.3 mb 2.6%
 File headers 199.3 kb 0.1%
 Complete size 238.9 mb 100.0%

并将Assets按大小降序列出：

16.0 mb 6.7% Built-in Texture2D:SpriteAtlasTexture-2048x2048-fmt33 
16.0 mb 6.7% Assets/Atlas/chapterUI.png 
16.0 mb 6.7% Assets/Atlas/PlayUI.png 
16.0 mb 6.7% Assets/Atlas/menu.png 
8.0 mb 3.3% Assets/Atlas/PlayUIEffects.png 
8.0 mb 3.3% Assets/Atlas/guideUI.png 
8.0 mb 3.3% Assets/Atlas/endUI.png 
8.0 mb 3.3% Assets/Atlas/PauseUI.png 
8.0 mb 3.3% Assets/Atlas/Rank.png 
5.7 mb 2.4% Assets/Images/UnitySprites/Backgrounds/MedicalCenter/MedicalCenter.png
 4.8 mb 2.0% Assets/Images/UnitySprites/Backgrounds/EiffelTower/EiffelTower.png 
4.0 mb 1.7% Assets/Images/UnitySprites/Backgrounds/StarsSky/Halo.png 
4.0 mb 1.7% Assets/Images/UnitySprites/Backgrounds/Moon/Moon.png
 4.0 mb 1.7% Assets/Atlas/PlayUISecond.png 
4.0 mb 1.7% Assets/Atlas/GiftPackage.png 
4.0 mb 1.7% Assets/Atlas/GameCenter.png 
4.0 mb 1.7% Assets/Atlas/GameBMFonts.png 
4.0 mb 1.7% Assets/Images/UnitySprites/Backgrounds/Freedom/Sun.png
 3.3 mb 1.4% Assets/Images/UnitySprites/Backgrounds/Taibei101/101Building.png 
2.9 mb 1.2% Assets/Images/UnitySprites/Backgrounds/Pyramid/Pyramid1.png 
2.8 mb 1.2% Assets/Images/UnitySprites/Backgrounds/Corcovado/Mountain.png .....

但在log中查找这些信息有些麻烦，于是想到开发一个插件来自动从Editor.log中分析并呈现结果

这个工具目前含有三个模块： 1.Assets Size Analyze 这个模块列出上面提到的资源大小相关信息，并且扫描StreamingAssets和Resources目录中的资源，方便查看是否可供优化 如果你使用了NGUI框架，并且sprite图集存放在Assets/Atlas目录，则会分析图集中的图片被当成assets再次打包的情况

2.Invalid Script Attach Analyze 列出无效的脚本引用，即抽出类似下面这种信息 Script attached to 'ShopContainer' in scene '' is missing or no valid script is attached.

3.Dll Reference Analyze 列出app引用的Dll

模块是可以扩展的，通过实现ILogAnalyzeModule来实现自定义模块，用LogAnalyzer.RegisterModule进行注册

这个工具开源了，github地址：
[https://github.com/xclouder/editor_log_analyze.git](https://github.com/xclouder/editor_log_analyze.git)