title: jenkins iOS自动构建
date: 2015-03-16 00:08:05
categories: iOS
tags: jenkins
---
网络上关于iOS自动构建的文章不少，有使用XCode Bot的，也有使用jenkins搭建的。
XCode Bot是苹果官方Server套件里面的一个组件，界面比较友好，“苹果味”比较浓，但是功能比较简单。
Jenkins是一个开源的CI Server解决方案，有数目众多的插件，不局限于iOS项目的构建，支持集群、任务管理。
我两种方法都使用了一段时间，最后还是选择了jenkins。
写这篇文章的原因是，网络上关于jenkins iOS的文章都比较过时，jenkins XCode Plugin很久没有更新了，对新版的XCode，打包成ipa的部分已经不可用。下面记录了搭建过程：

###jenkins的安装
网上资料太多，这里略

###jenkins项目设置
新建jenkins项目，为了后面配置方便，建议把项目名设置为XCode项目名称。比如XCode项目名为iOSTest，Xcode工程文件为iOSTest.xcproject，这里的项目名就填iOSTest  
![](/img/05031501.png)

我这里使用了Git仓库，在源码管理这里，设置好仓库地址以及相应的账号  
![](/img/05031502.png)

在“构建”步骤，添加一个“Execute shell”模块
![](/img/05031503.png)

添加下面的脚本  

```
if [ -d "${WORKSPACE}/builds" ]; then rm -rf ${WORKSPACE}/builds; fi;
mkdir ${WORKSPACE}/builds

xcodebuild -project ${WORKSPACE}/${JOB_NAME}.xcodeproj -scheme "${JOB_NAME}" archive -archivePath ${WORKSPACE}/builds/archive DEBUG_INFORMATION_FORMAT="dwarf"

xcodebuild -exportArchive -exportFormat IPA -archivePath ${WORKSPACE}/builds/archive.xcarchive -exportPath ${WORKSPACE}/builds/${JOB_NAME}.ipa -exportProvisioningProfile 'XC Ad Hoc: *'
```

DEBUG_INFORMATION_FORMAT="dwarf"表示不生成dsym文件，这样构建的时候会快一些。
[这里是xcodebuild命令详细的说明](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/xcodebuild.1.html)，大家可以根据需要添加，值得注意的是 -exportProvisioningProfile参数需要填写provision profile的名称，如“iOS Team ProvisioningProfile:*”



在“构建后操作”中，添加一个“Archive the artifacts”模块
![](/img/05031504.png)

用于存档的文件填写：builds/\*.\*
然后保存，Jenkins项目设置Done。

###XCode项目设置
在XCode项目中，需要把相应的Scheme设置为共享，并push修改到仓库。  
![](/img/05031505.png)  

![](/img/05031506.png)

把Release配置的证书和Provision Profile安装到CI Server上  
![](/img/05031507.png)

执行构建任务，如果亮蓝灯，恭喜你成功啦