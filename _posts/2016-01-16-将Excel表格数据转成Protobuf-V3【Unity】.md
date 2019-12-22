---
layout: post
title:  "将Excel表格数据转成Protobuf V3【Unity】"
date:   2016-01-16 17:17:01 +0800
categories: [Tool]
tags: [Tool,Protobuf,Unity]
---



版权声明：本文为Jumbo原创文章,采用[[知识共享 署名-非商业性使用-禁止演绎 4.0 国际 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)]，转载前请保证理解此协议
原文出处：https://www.jianshu.com/p/747c63d5d56b

### {{ page.title }}



## Win7 or Win10环境

### 安装Install.bat

- #### 工作原理流程


1、xls2protobuf_v3.py 将表格数据转成：*.proto *.bin *.txt等
2、protoc.exe 通过读取*.proto生成*.cs脚本解析类
3、Unity通过protobuf-V3.0\csharp_unity\Google.Protobuf插件读取*.bin文件

- #### 准备工作


  1、安装Python2.7,配置系统环境变量，如下图：
  2、切换到目录setuptools-18.7，执行 python setup.py install
  3、切换到目录xlrd-0.9.4，执行 python setup.py install
  4、切换到目录protobuf-V3.0\python，执行 python setup.py install

- #### 开始转换


  1、转换工具目录结构：
  -ConvertTools
     -xls          表格数据
     -xls2proto     通过表格数据生成*.proto(描述文件) *.bin(二进制) *.txt(明文)等
     -proto2cs      生成的*.cs文件
     -ConvertList.txt  批处理数据转换列表，配合ResConvertAll.bat
     -ResConvertAll.bat 执行批处理转换所有数据，读取ConverList.txt
     -ResConvert.bat 单个表格数据处理  命令格式：ResConvert [表格名] [xls文件名]
     -xls2protobuf_v3.py 表格数据转换工具，支持proto3最新格式 python xls2protobuf_v3.py [SHEET名称] [**.xls Excel表格] [PROTONAME 导出的名字] [option 0 生成proto和data 1 只生成proto 2 只生成data]  


2、数据表格制作: 参考xls2protobuf_v3.py定义描述
3、protoc.exe相关命令参数 : protoc --help查看 需要配置环境变量，在Path后面加上;{protobuf-V3.0\src\protoc.exe 路径目录}

## Unity使用

1、protobuf支持Unity的CSharp库：将protobuf-V3.0\csharp_unity\Google.Protobuf文件夹拷贝到Unity项目Plugins下面

2、proto生成的*.cs代码目录：Scripts\ResData
3、表格生成的*.bin数据目录：StreamingAssets\DataConfig

4、参考代码示例：   
using System;
using System.IO;
using UnityEngine;
using Google.Protobuf;

namespace Assets.Scripts.ResData
{
    class ResDataManager
    {

        private static ResDataManager sInstance;
    
        public static ResDataManager Instance
        {
            get
            {
                if (null == sInstance)
                {
                    sInstance = new ResDataManager();
                }
    
                return sInstance;
            }
        }


​        
​        public byte[] ReadDataConfig(string FileName)
​        {
​            FileStream fs = GetDataFileStream(FileName);
​            if (null != fs)
​            {
​                byte[] bytes = new byte[(int)fs.Length];
​                fs.Read(bytes, 0, (int)fs.Length);
​                
​                fs.Close();
​                return bytes;
​            }
​    
​            return null;
​        }


        private  FileStream GetDataFileStream(string fileName)
        {
            string filePath = GetDataConfigPath(fileName);
            if (File.Exists(filePath))
            {
                FileStream fs = new FileStream(filePath, FileMode.Open);
               
                return fs;
            }
    
            return null;
        }
        private string GetDataConfigPath(string fileName)
        {
            return Application.streamingAssetsPath + "/DataConfig/" + fileName;
        }
    }
}

读取二进制数据示例：
//GOODS_INFO_ARRAY对应的结构：GOODS_INFO
GOODS_INFO_ARRAY arr = GOODS_INFO_ARRAY.Parser.ParseFrom(ResDataManager.Instance.ReadDataConfig("Goods.bin"));
