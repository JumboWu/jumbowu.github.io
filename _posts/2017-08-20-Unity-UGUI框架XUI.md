---
layout: post
title:  "Unity-UGUI框架XUI"
date:   2017-08-20 07:09:27 +0800
categories: [Unity]
---



>版权声明：本文为Jumbo原创文章,采用[[知识共享 署名-非商业性使用-禁止演绎 4.0 国际 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)]，转载前请保证理解此协议
原文出处：[https://www.jianshu.com/p/99ff103b2b7d](https://www.jianshu.com/p/99ff103b2b7d)

## {{ page.title }}

![](http://upload-images.jianshu.io/upload_images/191918-3f08125a45a43267.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*****************************************************************
 A UI Framework based at Unity 5.6.2f1 with ugui, also you can use for ngui
 Jumbo @ 08/19/2017 Shanghai  
******************************************************************


## 简介
这是一个UI框架，基于Unity5.6.2f1,Demo中使用ugui，此框架也同样适用于ngui，稍加改动就可以了。


## 文件结构
>Assets
>>      -Prefab
>>>             App.prefab       App入口预设
>>>             UICanvas.prefab  UI画布预设
>>      -Scene
>>>             level.unity      测试场景
>>>             Start.unity      测试场景
>>      -Script
>>>             App.cs  程序入口脚本
>>>             -XTools        XTools工具集
>>>>                    MonoSingleton.cs MonoBehavior单例
>>>>                    Singleton.cs 普通单例
>>>             -XUI           XUI框架底层
>>>>                    BaseUI.cs    UI抽象类
>>>>                    UIBase.cs    UI基类
>>>>                    UIManager.cs UI管理器
>>      -UI               UI应用层
>>>             -Prefab         UI预设
>>>>                    -Resources    UI界面预设与脚本名对应
>>>>>                           UIMainMenu.prefab
>>>>>                           UITask.prefab
>>>             -Script         每个prefab对应一个以下同名的脚本，脚本基类UIBase
>>>>                    UIMainMenu.cs
>>>>                    UITask.cs


​        
##  演示
![](http://upload-images.jianshu.io/upload_images/191918-4064b8cc76203273.gif?imageMogr2/auto-orient/strip)

腾讯视频：https://v.qq.com/x/page/h0539q0iy3y.html

## 项目
Github：https://github.com/JumpWu/XUI
