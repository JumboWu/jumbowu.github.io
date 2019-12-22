---
layout: post
title:  "Lua-for-Windows"
date:   2016-03-11 11:37:05 +0800
categories: [Tool]
---

>版权声明：本文为Jumbo原创文章,采用[[知识共享 署名-非商业性使用-禁止演绎 4.0 国际 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)]，转载前请保证理解此协议
原文出处：[https://www.jianshu.com/p/d300fe106441](https://www.jianshu.com/p/d300fe106441)

## {{ page.title }}

本教程使用的Lua5.3.2

下载地址：http://www.lua.org/download.html

下载解压后：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/191918-f1ff37f0b9a3a62d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1、新建Build.bat
```

cd src
cl /O2 /W3 /c /DLUA_BUILD_AS_ALL l*.c
del lua.obj luac.obj
link /DLL /out:../bin/lua532.dll l*.obj
link /Lib /out:lua532.lib l*.obj

cl /O2 /W3 /c /DLUA_BUILD_AS_ALL lua.c luac.c
link /out:../bin/lua.exe lua.obj lua532.lib

del lua.obj
link /out:../bin/luac.exe l*.obj

del *.obj

cd ..
pause
```

2、编译Lua
cmd 运行 Build.bat

a、如果出现cl命令问题，请找到以x86 32为为例，如下cmd窗口：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/191918-7b3f1e6b2372ffa3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

b、cd 切换到Build.bat所在目录， 运行Build.bat， bin目录下生成：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/191918-e809d6962871f4a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好了，Lua编译器和解析器就生成了，大家可以愉快的学习Lua了。


【附件链接】: http://pan.baidu.com/s/1o6QUWUA 密码: 8bvp
