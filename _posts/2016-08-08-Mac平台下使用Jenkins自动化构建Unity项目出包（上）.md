---
layout: post
title:  "Mac平台下使用Jenkins自动化构建Unity项目出包（上）"
date:   2016-08-08 14:46:37 +0800
categories: [Tool]
---

>版权声明：本文为Jumbo原创文章,采用[[知识共享 署名-非商业性使用-禁止演绎 4.0 国际 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)]，转载前请保证理解此协议
原文出处：[https://www.jianshu.com/p/2dbd06ea554c](https://www.jianshu.com/p/2dbd06ea554c)

## {{ page.title }}

### 一、	安装JDK   jdk-8u91-macosx-x64.dmg

下载地址：
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

### 二、先下载android sdk for mac

给二个靠谱的网址：
 a. http://down.tech.sina.com.cn/page/45703.html
 b.http://mac.softpedia.com/get/Developer-Tools/Google-Android-SDK.shtml
下载后，解压到某个目录 

### 三、设置下载的代理服务器

命令行进入tools目录
然后输入 ./android sdk 请出SDK Manager的图形界面
Android SDK Manager -> Preferences...

![图一](http://upload-images.jianshu.io/upload_images/191918-76c6e5986ab1be6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


http proxy server这里填写： mirrors.neusoft.edu.cn （感谢东软搭建国内的镜像服务器，为广大程序员造福无数）
端口填写80，然后把Force https:// 前的勾勾上

##四、mac顶部菜单Tools->Manage Add-on Site

![图二](http://upload-images.jianshu.io/upload_images/191918-6cf75714cd2c6c2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把下面这堆网址：
http://mirrors.neusoft.edu.cn/android/repository/addon-6.xml 
http://mirrors.neusoft.edu.cn/android/repository/addon.xml 
http://mirrors.neusoft.edu.cn/android/repository/extras/intel/addon.xml 
http://mirrors.neusoft.edu.cn/android/repository/sys-img/android-tv/sys-img.xml 
http://mirrors.neusoft.edu.cn/android/repository/sys-img/android-wear/sys-img.xml 
http://mirrors.neusoft.edu.cn/android/repository/sys-img/android/sys-img.xml 
http://mirrors.neusoft.edu.cn/android/repository/sys-img/google_apis/sys-img.xml 
http://mirrors.neusoft.edu.cn/android/repository/sys-img/x86/addon-x86.xml 
http://mirrors.neusoft.edu.cn/android/repository/addons_list-2.xml 
http://mirrors.neusoft.edu.cn/android/repository/repository-10.xml
全手动New加进去，然后就可以下载了

![图三](http://upload-images.jianshu.io/upload_images/191918-14fef8aafc7c5f76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注：上图中加圈的项，建议勾上，否则有可能创建不了Android模拟设备

3、	安装Ant 1.9.4
下载地址：http://ant.apache.org/bindownload.cgi

![图四](http://upload-images.jianshu.io/upload_images/191918-e6c677a60a988bca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


4、	安装Python 2.7
下载地址：https://www.python.org/downloads/mac-osx/
Mac系统自带Python，通过命令行 python可查看具体信息
5、	安装Unity 4.6.5p4
下载地址：https://unity3d.com/cn/unity/qa/patch-releases
6、	安装Jenkins 2.7.1
下载地址：https://jenkins.io/
7、	Unity 3D Android SDK及JDK路径

![图五](http://upload-images.jianshu.io/upload_images/191918-c56bb6027db29e0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

8、	环境变量配置
        一般在/etc/bashrc中配置环境变量。 全局（公有）配置，bash shell执行时，不管是何种方式，都会读取此文件。（刚开始还以为是ant下面的etc呢，找了半天也没有bashrc文件，最后才知道原来是home下面的）
          ①、获取root权限
             sudo -s   
            根据提示输入密码，输入没有显示不要在意，输入完毕回车键结束即可，提示符会变成bash-3.2#（如图所示）

![图六](http://upload-images.jianshu.io/upload_images/191918-84dfa737f99c1ed7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

         ②、修改bashrc的读写权限
             chmod +w /etc/bashrc
         ③、修改bashrc文件
            vi /etc/bashrc
        (②、③步如图所示)

![图七](http://upload-images.jianshu.io/upload_images/191918-838740aa89bb9093.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

         （/etc/bashr命令行主要是我验证bashrc的路径而已，没有实际意义）
       ④、进入bashrc文件的编辑状态，在最后添加如图2句话即可（在输入完第③步的命令后立刻弹出）

![图八](http://upload-images.jianshu.io/upload_images/191918-beec7f6f7f52ae45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


注意书写格式，第一次的时候在等号的左右多输入2个空格都没有配置成功的说
        ⑤、按ESC键退出编辑状态。输入:wq!保存并退出。
        ⑥、测试是否安装成功
             重新打开终端，输入ant -version。如果成功会显示如图信息

![图九](http://upload-images.jianshu.io/upload_images/191918-16ec36d2df416210.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

     我的ant终于安装成功了，想说命令行真的很强大啊，就在安装Ant的时候查了一点Mac命令行的一些简单命令，分享一下。
     1、获取权限            sudo -s
     2、列出文件            ls 参数名 目录名    （参数 -w 显示中文，-l 详细信息， -a 包括隐藏文件）例如：ls /System/Library/Extensions (可以不写参数)
     3、转换目录            cd
     4、新建目录            mkdir 目录名
     5、拷贝文件            cp 参数 源文件 目标文件（参数R表示对目录进行递归操作）
     6、删除文件            rm 参数 文件（参数－rf 表示递归和强制，千万要小心使用，如果执行了 rm -rf / 你的系统就全没了）
     7、移动文件            mv 文件
     8、更改文件权限      chmod 参数 权限 文件
     9、更改文件属主      chown 参数 用户:组 文件
     10、文本编辑           nano 文件名（编辑完成后 用 Ctrl ＋O 存盘，Ctrl＋X 退出，另一个文本编辑软件是 vi，操作有些古怪，熟了是非常好用的，而且在所有类Unix系统中用
   小技巧：用 Tab 键自动补齐命令


需要配置的环境变量（jenkins环境变量也需要配置）：
```
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_92.jdk/Contents/Home
export JAVA_HOME

ANDROID_HOME=/Users/build/work/android-sdk-macosx
export ANDROID_HOME

ANDROID_SDK_HOME=/Users/build/work/android-sdk-macosx
export ANDROID_SDK_HOME

ANDROID_SDK_ROOT=/Users/build/work/android-sdk-macosx
export ANDROID_SDK_ROOT

ANT_HOME=/Users/build/work/apache-ant-1.9.7
export ANT_HOME

PATH=${PATH}:${JAVA_HOME}/bin:${JAVA_HOME}/jre/bin:${JAVA_HOME}/bin:${ANT_HOME}/bin:${ANDROID_SDK_HOME}/platform-tools:${ANDROID_SDK_HOME}/tools
export PATH

CLASSPATH=${CLASSPATH}:${JAVA_HOME}/libdt.jar:${JAVA_HOME}/lib/tools.jar:${JAVA_HOME}/lib/annotations.jar
export CLASSPATH
```

Jenkins 安装配置将在下一篇介绍...
