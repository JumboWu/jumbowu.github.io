---
layout: post
title:  "NGUI-表情支持控件UIEmojiLabel，支持自适应多行多列"
date:   2017-03-03 17:27:44 +0800
categories: [Unity]
---



>版权声明：本文为Jumbo原创文章,采用[[知识共享 署名-非商业性使用-禁止演绎 4.0 国际 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)]，转载前请保证理解此协议
原文出处：[http://www.jianshu.com/p/a604b3f1fce3](http://www.jianshu.com/p/a604b3f1fce3)

## {{  page.title }}

NGUI出来已久，使用范围，场景越来越广。本身在高版本NGUI也支持表情，但存在一些缺陷，例如：支持文字和表情显示，但要求是必须文字和表情都需要在BMFont制作，对于文字量非常大的需求下，NGUI自带的文本混合表情，就显得力不从心。具体用过的同学，应该都知道的。

NGUI控件扩展，支持自适应多行多列表情
思路：
1、输入文本表情，在文本处理的时候，处理表情字串，记录位置
2、根据位置信息，得到文本在渲染时候的顶点位置，存在this.geometry.verts中
3、为什么位置要*4？ 因为每个字串有4个顶点数据
4、把站位的emSpace ，位置赋给UISprite（显示表情的控件）
5、所有的表情，通过表情管理器动态统一管理，不显示时，回收

1、UIEmojiLabel
```
/*
 * Emoji Label @By Jumbo 2017/3/3
 * */

using UnityEngine;
using System.Collections;
using System.Collections.Generic;
using System;
using System.Text;

public class UIEmojiLabel : UILabel {
    
    private  char emSpace = '\u2001';
    private List<GameObject> mEmojiList = new List<GameObject>();

    public UIAtlas emAtlas = null;
    protected override void OnStart()
    {
        base.OnStart();

//表情统一管理器，只初始化一次
        UIEmojiWrapper.Instance.Init(emAtlas);
    }

    //自动转码
    public override void OnFill (BetterList<Vector3> verts, BetterList<Vector2> uvs, BetterList<Color32> cols)
    {
        base.OnFill(verts, uvs, cols);


        //Test  测试代码,正式使用，这里注释掉，外面改变Label 的值
        string tx = "zhongz中国" + UIEmojiWrapper.Instance.GetConvertedString("1f61c") + UIEmojiWrapper.Instance.GetConvertedString("1f4fa") + UIEmojiWrapper.Instance.GetConvertedString("1f63f") + UIEmojiWrapper.Instance.GetConvertedString("1f607") + "国家回忆中心年内" + "sdfjsdlkj" + UIEmojiWrapper.Instance.GetConvertedString("1f63b") + "国家" + UIEmojiWrapper.Instance.GetConvertedString("1f467-1f3ff") + UIEmojiWrapper.Instance.GetConvertedString("1f450-1f4ff");// + "好行。天使good bye";
        text = tx;
///~测试代码
        StartCoroutine(SetUITextThatHasEmoji(text));
    }


    private struct PosStringTuple
    {
        public int pos;
        public string emoji;

        public PosStringTuple(int p, string s)
        {
            this.pos = p;
            this.emoji = s;
        }
    }


    public IEnumerator SetUITextThatHasEmoji(string inputString)
    {
        List<PosStringTuple> emojiReplacements = new List<PosStringTuple>();
        StringBuilder sb = new StringBuilder();

        //先回收
        UIEmojiWrapper.Instance.PushEmoji(ref mEmojiList);
        int i = 0;
        while (i < inputString.Length)
        {
            string singleChar = inputString.Substring(i, 1);
            string doubleChar = "";
            string fourChar = "";

            if (i < (inputString.Length - 1))
            {
                doubleChar = inputString.Substring(i, 2);
            }

            if (i < (inputString.Length - 3))
            {
                fourChar = inputString.Substring(i, 4);
            }

            if (UIEmojiWrapper.Instance.HasEmoji(fourChar))
            {
                // Check 64 bit emojis first
                sb.Append(emSpace);
                emojiReplacements.Add(new PosStringTuple(sb.Length - 1, fourChar));
                i += 4;
            }
            else if (UIEmojiWrapper.Instance.HasEmoji(doubleChar))
            {
                // Then check 32 bit emojis
                sb.Append(emSpace);
                emojiReplacements.Add(new PosStringTuple(sb.Length - 1, doubleChar));
                i += 2;
            }
            else if (UIEmojiWrapper.Instance.HasEmoji(singleChar))
            {
                // Finally check 16 bit emojis
                sb.Append(emSpace);
                emojiReplacements.Add(new PosStringTuple(sb.Length - 1, singleChar));
                i++;
            }
            else
            {
                sb.Append(inputString[i]);
                i++;
            }
        }

        // Set text
        this.text = sb.ToString();

        yield return null;

      
        for (int j = 0; j < emojiReplacements.Count; j++)
        {
            int emojiIndex = emojiReplacements[j].pos;
            //表情替换,计算位置,大小
            GameObject go = UIEmojiWrapper.Instance.PopEmoji();
            if (go != null)
            {
                UISprite spt = go.GetComponent<UISprite>();
                if (spt != null)
                {
                    string emoji = UIEmojiWrapper.Instance.GetEmoji(emojiReplacements[j].emoji);
                    if (!string.IsNullOrEmpty(emoji))
                    {
                        spt.name = emoji;
                        spt.spriteName = emoji;
                    }
//mPrintedSize 父类权限私有，可以改为子类可范围的保护的权限，渲染的字体大小变化，来改变UISprite的大小
                    spt.width = this.mPrintedSize;
                    spt.height = this.mPrintedSize;
                    spt.transform.parent = this.transform;
                    spt.transform.localScale = Vector3.one;
                    spt.transform.localPosition = new Vector3(this.geometry.verts[emojiIndex * 4].x + spt.width / 2, this.geometry.verts[emojiIndex * 4].y + spt.height / 2);
                }

                mEmojiList.Add(go);
            }
        }
}

}

```

2、UIEmojiLabelInspector 编辑器监视
```
/*
 * Emoji Label Inspector @By Jumbo 2017/3/3
 * */

#if !UNITY_3_5 && !UNITY_FLASH
#define DYNAMIC_FONT
#endif

using UnityEngine;
using UnityEditor;

/// <summary>
/// Inspector class used to edit UILabels.
/// </summary>

[CanEditMultipleObjects]
#if UNITY_3_5
[CustomEditor(typeof(UILabel))]
#else
[CustomEditor(typeof(UILabel), true)]
#endif
public class UIEmojiLabelInspector : UILabelInspector
{
	
		
	/// <summary>
	/// Draw the label's properties.
	/// </summary>

	protected override bool ShouldDrawProperties ()
	{
        bool isValid = base.ShouldDrawProperties();

        EditorGUI.BeginDisabledGroup(!isValid);
        NGUIEditorTools.DrawProperty("Atlas", serializedObject, "emAtlas");//表情所在的图集Atlas
        EditorGUI.EndDisabledGroup();
        return isValid;
	}
}

```

3、UIEmojiWrapper 表情统一管理器
```
/*
 * Emoji Wrapper @By Jumbo 2017/3/3
 * */

using System;
using System.Collections.Generic;
using System.Text;
using UnityEngine;

class UIEmojiWrapper
{
    private bool hasAtlas = false;
    private UIAtlas mAtlas;
    private GameObject emojiPrefab;
    private Dictionary<string, string> emojiName = new Dictionary<string, string>();
    private Queue<GameObject> freeSprite = new Queue<GameObject>();
    
    private List<GameObject> usedSprite = new List<GameObject>();

    private Vector3 OutOffScreen = new Vector3(10000f, 10000f, 10000f);

    private static UIEmojiWrapper sInstance;
    public static UIEmojiWrapper Instance
    {
        get
        {
            if(sInstance == null)
            {
                sInstance = new UIEmojiWrapper();
            }
            
            return sInstance;
        }
    }
    
    public void Init(UIAtlas atlas)
    {
        if (!hasAtlas)
        {
            if (atlas == null)
            {
                LogUtil.LogError(WXLogTag.LogTag_UI, "[UIEmojiWrapper Atlas is null]");
                return;
            }
            mAtlas = atlas;
            //预分配
            for (int i = 0, cnt = mAtlas.spriteList.Count; i < cnt; i++)
            {
                emojiName.Add(GetConvertedString(mAtlas.spriteList[i].name), mAtlas.spriteList[i].name);
            }

            emojiPrefab = new GameObject();
            UISprite spt = emojiPrefab.AddComponent<UISprite>();
            if (spt != null)
            {
                spt.transform.localPosition = OutOffScreen;
                spt.name = "emojiPrefab";

            }

            hasAtlas = true;
        }
        
    }

    public string GetConvertedString(string inputString)
    {
        string[] converted = inputString.Split('-');
        for (int j = 0; j < converted.Length; j++)
        {
            converted[j] = char.ConvertFromUtf32(Convert.ToInt32(converted[j], 16));
        }
        return string.Join(string.Empty, converted);
    }

    public string GetEmoji(string encode)
    {
        string em;

        if(emojiName.TryGetValue(encode, out em))
        {
            return em;
        }

        return null;
    }
    public bool HasEmoji(string key)
    {
        return emojiName.ContainsKey(key);
    }

     public void OnPostFill (UIWidget widget, int bufferOffset, BetterList<Vector3> verts, BetterList<Vector2> uvs, BetterList<Color32> cols)
     {
         if (widget != null)
         {
             if(!widget.isVisible)
             {
                 UISprite spt = widget as UISprite;
                 if (spt != null)
                 {
                     PushEmoji(spt.gameObject);
                 }
             }
         }
     }

    public GameObject PopEmoji()
    {
        if (freeSprite.Count <= 0)
        {
            if (emojiPrefab == null)
                return null;

            GameObject tran = GameObject.Instantiate(emojiPrefab) as GameObject;
            UISprite spt = tran.GetComponent<UISprite>();
            if (spt != null)
            {
                spt.atlas = mAtlas;
                spt.hideIfOffScreen = true;
                spt.onPostFill += OnPostFill;
            }
                

            freeSprite.Enqueue(tran);
        }

        GameObject sptRet = freeSprite.Dequeue();
        
        if (sptRet != null)
        {
            usedSprite.Add(sptRet);
        }

        return sptRet;

    }


    public void PushEmoji(GameObject spt)
    {
        spt.transform.localPosition = OutOffScreen;
        spt.transform.parent = null;
        freeSprite.Enqueue(spt);
    }

    public void PushEmoji (ref List<GameObject> list)
    {
        for (int i = 0, cnt = list.Count; i < cnt; i++)
        {
            PushEmoji(list[i]);
        }

        list.Clear();
    }
}

```

有更好的方案，不吝赐教~~

【原创】转摘请保留原文链接http://www.jianshu.com/p/a604b3f1fce3
Demo效果：
![文本表情混排效果](http://upload-images.jianshu.io/upload_images/191918-a78ddad043f1d7ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
