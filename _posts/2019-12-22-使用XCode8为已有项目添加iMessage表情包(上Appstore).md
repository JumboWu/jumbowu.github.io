---
layout: post
title:  "使用XCode8为已有项目添加iMessage表情包(上Appstore)"
date:   2019-12-22 10:29:15 +0800
categories: [iOS]
---

>版权声明：本文为Jumbo原创文章,采用[[知识共享 署名-非商业性使用-禁止演绎 4.0 国际 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)]，转载前请保证理解此协议
原文出处：[https://www.jianshu.com/p/6329737d26b3](https://www.jianshu.com/p/6329737d26b3)

大多说情况下，我们开发项目的前期并没有集成iMessage功能的需求，然而在项目后期，产品应用即将上线前，需要为应用添加一些新的功能，例如：3D Touch, Replay kit,iMessage等功能，这样比较容易获得苹果推荐。所以接下来，我们会讲到，如何利用XCode8 为现有的项目添加iMessage表情包，并且提交上Appstore。

1、XCode8 打开已有项目
2、添加Application Extension

![添加Target](http://upload-images.jianshu.io/upload_images/191918-27d4e1edaea41cbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3、在打开的窗口中，选择Sticker Pack Extension->Next

![Sticker Pack Extension](http://upload-images.jianshu.io/upload_images/191918-7eccaafd0ff135c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4、配置贴纸表情包Target->Finish
![贴纸表情包Target](http://upload-images.jianshu.io/upload_images/191918-533a72e89415bc25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5、在工程目录的左边，可以发现，多出来一个sticker文件夹


![sticker](http://upload-images.jianshu.io/upload_images/191918-5ee61b32a7930dc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6、切换Target到刚才新加的sticker Target

![switch to sticker target](http://upload-images.jianshu.io/upload_images/191918-8b9b3ac677860592.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

7、修改贴纸表情包的配置
![贴纸表情包配置](http://upload-images.jianshu.io/upload_images/191918-d2ad638a4ee987c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

8、添加iMessage App Icon

![iMessage App Icon](http://upload-images.jianshu.io/upload_images/191918-40cc203ecc7c8b84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

9、添加Sticker Pack表情

![贴纸表情](http://upload-images.jianshu.io/upload_images/191918-1b7d889c22ab3e8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

10、测试
打开iMessage ->Store->管理->打开iMessage应用,可以开始测试贴纸表情了


***需要注意***
i、贴纸表情包的BundleID，必须是主BundleID+sticker，这个sticker名字可以自己定义；
ii、贴纸表情包的显示名称，与主产品一致；
iii、签名必须要Automatically manager signing,否则打包会出错，也上传不了Appstore；
