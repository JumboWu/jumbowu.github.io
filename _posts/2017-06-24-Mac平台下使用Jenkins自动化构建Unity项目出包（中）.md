---
layout: post
title:  "Mac平台下使用Jenkins自动化构建Unity项目出包（中）"
date:   2017-06-24 11:16:42 10:29:15 +0800
categories: [Tool]
---

>版权声明：本文为Jumbo原创文章,采用[[知识共享 署名-非商业性使用-禁止演绎 4.0 国际 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)]，转载前请保证理解此协议
原文出处：[https://www.jianshu.com/p/41a3b72a4d36](https://www.jianshu.com/p/41a3b72a4d36)

## {{ page.title }}

Mac平台下使用Jenkins自动化构建Unity项目出包（上）篇已经介绍了部署的前期工作，接下来就介绍下在Jenkins中如何配置自动化参数，来自动编译项目。

1、在浏览器中输入Jenkins访问URL地址，建议用Chrome
2、首次登录Jenkins会要求，按步骤就行
3、进入Jenkins，
![Jenkins首页](http://upload-images.jianshu.io/upload_images/191918-f73fd4bc48f1af94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4、先添加一个项目文件夹，之后关于这个项目的所有版本，可以在里面新建项目
5、点击上面新建的项目文件下，进入项目列表
![项目列表](http://upload-images.jianshu.io/upload_images/191918-fe4b6bb41c21ce07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
6、新建项目-点击左边的【New Item】进入项目配置
![项目设置](http://upload-images.jianshu.io/upload_images/191918-0d0020d5d49c63e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
7、构建项目设置-如下：

![项目描述](http://upload-images.jianshu.io/upload_images/191918-0567d9a73c58bee9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
8、参数设置

![参数设置-1](http://upload-images.jianshu.io/upload_images/191918-c6e55376600aacca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![参数设置-2](http://upload-images.jianshu.io/upload_images/191918-54eebe5d7ec978c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![参数设置-3](http://upload-images.jianshu.io/upload_images/191918-94cfd6d32f30e2f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![构建环境](http://upload-images.jianshu.io/upload_images/191918-3c8de4d51efee1e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![构建脚本](http://upload-images.jianshu.io/upload_images/191918-b00a0b2a4bdd0abd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Jenkins上的配置基本完成了，后期可以根据项目的需求，灵活的做出改动了，比如添加参数，修改Shell命令等。Shell脚本编写，调用Unity管道打包，将在下一篇文章介绍。




