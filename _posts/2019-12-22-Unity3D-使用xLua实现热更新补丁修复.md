---
layout: post
title:  "Unity3D 使用xLua实现热更新补丁修复"
date:   2017-01-25 11:14:53 +0800
categories: [Unity]
---

>版权声明：本文为Jumbo原创文章,采用[[知识共享 署名-非商业性使用-禁止演绎 4.0 国际 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)]，转载前请保证理解此协议
原文出处：[http://www.jianshu.com/p/dc4de5612d9e ](http://www.jianshu.com/p/dc4de5612d9e )

## {{  page.title }}

在Unity3D项目中，逻辑代码热更新这一块，现在有很多实现解决方案，基本都是借助Lua来实现的，在这众多之中，最后还是选择xLua，最早了解xLua是在腾讯的手游项目中，腾讯Apollo通用组件中，只是涉及的项目中，并没有xLua的源码及文档相关的说明。可能当时xLua还在改进当中。直到2017年1月3日，腾讯Github开源项目中，出现[xLua](https://github.com/Tencent/xLua/)的身影。xLua优势：1、集成快 (几乎不用改变原本项目的代码) 2、专职人员负责维护（企鹅还是有这个实力·~~·），说再多也不如自己切身体验一番。下面就帮大家，快速入门下。

一、xLua源码: https://github.com/Tencent/xLua/ 没有帐号或者下载不了的同学，可以邮箱留言，发一份给大伙；

二、获取到源码后，首先认真阅读[README.md](https://github.com/Tencent/xLua/blob/master/README.md)内容，可以更好的帮大家了解xLua，是否项目中遇到的问题，可以通过xLua来帮忙处理？；

三、根据提供的示例：
[01_Helloworld](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Examples/01_Helloworld): 快速入门的例子。
[02_U3DScripting](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Examples/02_U3DScripting): 展示怎么用lua来写MonoBehaviour。
[03_UIEvent](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Examples/03_UIEvent): 展示怎么用lua来写UI逻辑。
[04_LuaObjectOrented](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Examples/04_LuaObjectOrented): 展示lua面向对象和C#的配合。
[05_NoGc](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Examples/05_NoGc): 展示怎么去避免值类型的GC。
[06_Coroutine](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Examples/06_Coroutine): 展示lua协程怎么和Unity协程相配合。
[07_AsyncTest](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Examples/07_AsyncTest): 展示怎么用lua协程来把异步逻辑同步化。
[08_Hotfix](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Examples/08_Hotfix): 热补丁的示例（需要开启热补丁特性，如何开启请看[指南](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/hotfix.md)）。
配合文档：
[XLua教程.doc](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/XLua%E6%95%99%E7%A8%8B.doc)：教程，其配套代码[这里](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Tutorial)。
[XLua的配置.doc](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/XLua%E7%9A%84%E9%85%8D%E7%BD%AE.doc)：介绍如何配置xLua。
[XLua增加删除第三方lua库.doc](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/XLua%E5%A2%9E%E5%8A%A0%E5%88%A0%E9%99%A4%E7%AC%AC%E4%B8%89%E6%96%B9lua%E5%BA%93.doc)：如何增删第三方lua扩展库。
[XLua API.doc](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/XLua_API.doc)：API文档。
可以更好的了解xLua具备那些特性，如何便捷使用。

四、最重要的功能：热补丁
xLua支持热补丁，这意味着你可以：
1、开发只用C#；
2、运行也是C#，性能可以秒杀lua；
3、出问题了才用Lua来改掉C#出问题的部位，下次整体更新时换回正确的C#；能做到用户不重启程序fix bug；

如果你仅仅希望用热更新来fix bug，这是强烈建议的做法。[这里](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/hotfix.md)是使用指南。
这个文档说明一定要仔细阅读https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/hotfix.md

很多刚开始接触的同学，因为没有认真看文档说明，容易出现一些问题，这边列举下：
注意：a、HOTFIX_ENABLE宏是否在Unity3D的File->Build Setting->Scripting Define Symbols下添加
b、Mono.Cecil.*是否拷贝：OSX命令行 cp /Applications/Unity/Unity.app/Contents/Managed/Mono.Cecil.* Project/Assets/XLua/Src/Editor/

Win命令行 copy UnityPath\Editor\Data\Managed\Mono.Cecil.* Project\Assets\XLua\Src\Editor\

c、菜单栏中，执行xLua/Generat Code, 会在新建Gen文件夹，下面生成一些wrap文件

![](http://upload-images.jianshu.io/upload_images/191918-67ec62ca1c178eaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
具体生成那些类型对应的wrap文件，需要自己配置，详见：https://github.com/Tencent/xLua/blob/master/Assets/XLua/Examples/ExampleGenConfig.cs
在项目中集成，最好自己写一份配置，替换ExampleGenConfig，比如取名CustomGenConfig.cs，加入了[Hotfix]标签
```
  //需要热更新的类型
	[Hotfix]
	public static List<Type> by_field = new List<Type>()
	{

	};
```
d、要等打印了hotfix inject finish!后才运行例子，否则会类似xlua.access, no field __Hitfix0_Update的错误

五、整合xLua进项目
1、
![](http://upload-images.jianshu.io/upload_images/191918-7dcd9375635c4273.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、

![](http://upload-images.jianshu.io/upload_images/191918-47cbb5a4c0f56756.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3、

![](http://upload-images.jianshu.io/upload_images/191918-1519ddc7a06ecb2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

先贴上代码（myHotfix.lua 编写fixbug逻辑）：
```
using UnityEngine;
using System.Collections;
using XLua;

/// <summary>
/// 先判断更新热补丁文件，然后再执行Fixing
/// </summary>
public class xLuaInstance : MonoSingleton<xLuaInstance>{

    LuaEnv luaenv;

    const string fixFile = "myHotfix.lua";

    public string FixFilePath
    {
        get
        {
            return FileUtils.GetFullPath(null, fixFile, FileUtils.StorageType.Persistent);
        }
    }

    /// <summary>
    /// 热补丁修复中。。。
    /// </summary>
    public void Fixing()
    {

        byte[] bytes = System.IO.File.ReadAllBytes(FixFilePath);

        if (bytes != null)
        {
            luaenv = new LuaEnv();
            luaenv.AddLoader((ref string filename) =>
            {
                if (filename == "myHotfix")
                {
                    filename = FixFilePath;
                    return bytes;
                }

                return null;
            });

            luaenv.DoString(@"require 'myHotfix'");
        }
    }

    void Update()
    {
        if (luaenv != null)
            luaenv.Tick();
    }

    void OnDestroy()
    {
        if (luaenv != null)
            luaenv.Dispose();
    }
}
```


4、大致流程
i、游戏一开始启动时，检查远程服务器上是否有新的myHotfix.lua文件（例如：md5比对，自己加解密），有的话，下载下来，放在指定目录，没有的话，读取本地已有的myHotfix.lua文件，若文件不存在，则说明不需要热修复，一般这种情况是在项目刚发布的早起，没有进行打补丁；

ii、项目发布版本前，需要在CustomGenConfig.cs中 加入需要添加[Hotfix]标签的类型，想要更灵活使用的话，可以自己写个配置表，读取配置中的类型，自动添加，***只有加入[Hotfix]标签的类型，才可以调用xlua.hotfix()
![](http://upload-images.jianshu.io/upload_images/191918-5480a71b1579ea88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
,不然会报错，切记！！！

iii、如果你的项目是自动化发布版本，需要在调用打包之前，生成wrap， 调用这个CSObjectWrapEditor.Generator.GenAll();


####MonoSingleton.cs
``` 
//非线程安全
using UnityEngine;

    /// <summary>
    ///     基类继承树中有MonoBehavrour类的单件实现，这种单件实现有利于减少对场景树的查询操作
    /// </summary>
    /// <typeparam name="T"></typeparam>
    public class MonoSingleton<T> : MonoBehaviour where T : Component
    {
        // 单件子类实例
        private static T _instance;

        // 在单件中，每个物件的destroyed标志设计上应该分割在不同的存储个空间中，因此，忽略R#的这个提示
        // ReSharper disable once StaticFieldInGenericType
        private static bool _destroyed;

        /// <summary>
        ///     获得单件实例，查询场景中是否有该种类型，如果有存储静态变量，如果没有，构建一个带有这个component的gameobject
        ///     这种单件实例的GameObject直接挂接在bootroot节点下，在场景中的生命周期和游戏生命周期相同，创建这个单件实例的模块
        ///     必须通过DestroyInstance自行管理单件的生命周期
        /// </summary>
        /// <returns>返回单件实例</returns>
        public static T Instance
        {
            get{

                 if (_instance == null && !_destroyed)
                 {
                     _instance = (T) FindObjectOfType(typeof (T));
                     if (_instance == null)
                    {
                       GameObject go = new GameObject(typeof (T).Name);
                       _instance =  go.AddComponent<T>();

                        DontDestroyOnLoad(_instance);

                       var singletonRootGo = GameObject.Find("MonoSingletonRoot");
                       if (singletonRootGo != null)
                       {
                           go.transform.parent = singletonRootGo.transform;
                         }
                    }
                 }
                return _instance;
            }
           
         }

        /// <summary>
        ///     删除单件实例,这种继承关系的单件生命周期应该由模块显示管理
        /// </summary>
        public static void DestroyInstance()
        {
            if (_instance != null)
                Destroy(_instance.gameObject);

            _destroyed = true;
            _instance = null;
        }

        /// <summary>
        ///     Awake消息，确保单件实例的唯一性
        /// </summary>
        protected virtual void Awake()
        {
            if (_instance != null && _instance.gameObject != gameObject) Destroy(gameObject);
            else if(_instance == null)
                _instance = GetComponent<T>();

            DontDestroyOnLoad(_instance);
        }

        /// <summary>
        ///     OnDestroy消息，确保单件的静态实例会随着GameObject销毁
        /// </summary>
        protected virtual void OnDestroy()
        {
            if (_instance != null && _instance.gameObject == gameObject)
            {
                _instance = null;                
            }
        }

        /// <summary>
        ///     Have Instance
        /// </summary>
        /// <returns></returns>
        public static bool HaveInstance()
        {
            return _instance != null;
        }
    }
```
如何调试Lua脚步，接下来，我会写一篇关于ZeroBrane这个工具。

在此感谢xLua作者：车雄生 chexiongsheng！！！
期待大家的支持！！！
祝大家在17年万事如意，身体健康！！！
