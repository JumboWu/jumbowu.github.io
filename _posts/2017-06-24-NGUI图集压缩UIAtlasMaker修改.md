---
layout: post
title:  "NGUI图集压缩UIAtlasMaker修改"
date:   2017-06-24 16:59:30 +0800
categories: [Unity]
---

>版权声明：本文为Jumbo原创文章,采用[[知识共享 署名-非商业性使用-禁止演绎 4.0 国际 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)]，转载前请保证理解此协议
原文出处：[https://www.jianshu.com/p/2d41e280deb7](https://www.jianshu.com/p/2d41e280deb7)

## {{ page.title }}

阅读前请先了解下
[Unity3D 图片纹理格式](http://www.jianshu.com/p/b9a6feb6d7b5) 
[Unity游戏开发图片纹理压缩方案](http://www.jianshu.com/p/0f09206760b1)

* 解决的问题？
 * 包体过大
 * 内存占用大
 * 优化性能

* 方案如何实现
  * 通过修改NGUI UIAtalsMaker图集打包工具来修改图集平台对应最优格式

 * Android： 转成两张RGB ETC 4 bits图，一张保存RGB，一张保存Alpha
 * iOS ：一般情况下转成RGBA PVRTC 4 bits就行，转成两张RGB PVRTC 4 bits图，一张保存RGB，一张保存Alpha精度更高，工具中转成的是两张图

  * 先目睹下

![Atlas Maker](http://upload-images.jianshu.io/upload_images/191918-1d9ab0241edcdc1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   UIOptimization: 图集优化选项
   Force Square：图集图片方形 长宽相等 2的n次方
   尽量图片大小小于等于2048
*  RGB+Alpha合并渲染 复制 NGUI\Resources\Shaders\Unlit - Transparent Colored.shader重命名Unlit - Transparent Colored Split.shader


```
//暴露在Unity编辑器，赋值资源、改动
    	Properties {
		     _MainTex ("Base (RGB)", 2D) = "black" {} //RGB
		       _MaskTex ("Alpha (A)", 2D) = "black" {} //Alpha
	     }

       Pass{

       //添加
       sampler2D _MaskTex;//与Properties同名贴图属性绑定

       //修改
       fixed4 frag (v2f IN) : COLOR{
				fixed4 texClr = tex2D(_MainTex, IN.texcoord);
				texClr.a = tex2D(_MaskTex, IN.texcoord).r; //顶点Alpah值
				
				return texClr * IN.color;
			}

           }


```

* 修改UIAtalsMaker.cs 

  //OnGUI()中添加

```
GUILayout.BeginHorizontal();
NGUISettings.uiOptimization = EditorGUILayout.Toggle("UIOptimization", NGUISettings.uiOptimization, GUILayout.Width(100f));
GUILayout.Label("Split texture with 2 RGB ETC 4 (and Alpha) Android ,iOS 2 RGB PVRTC4", GUILayout.MinWidth(70f));
        GUILayout.EndHorizontal();
```

//修改实现 static public bool UpdateTexture (UIAtlas atlas, List<SpriteEntry> sprites)

```
static public bool UpdateTexture (UIAtlas atlas, List<SpriteEntry> sprites)
	{
        bool bUIOptimization = NGUISettings.uiOptimization;
		// Get the texture for the atlas
		Texture2D tex = atlas.texture as Texture2D;
		string oldPath = (tex != null) ? AssetDatabase.GetAssetPath(tex.GetInstanceID()) : "";
		string newPath = NGUIEditorTools.GetSaveableTexturePath(atlas);

		// Clear the read-only flag in texture file attributes
		if (System.IO.File.Exists(newPath))
		{
#if !UNITY_4_1 && !UNITY_4_0 && !UNITY_3_5
			if (!AssetDatabase.IsOpenForEdit(newPath))
			{
				LogUtil.LogError(newPath + " is not editable. Did you forget to do a check out?");
				return false;
			}
#endif
			System.IO.FileAttributes newPathAttrs = System.IO.File.GetAttributes(newPath);
			newPathAttrs &= ~System.IO.FileAttributes.ReadOnly;
			System.IO.File.SetAttributes(newPath, newPathAttrs);
		}

		bool newTexture = (tex == null || oldPath != newPath);

		if (newTexture)
		{
			// Create a new texture for the atlas
			tex = new Texture2D(1, 1, TextureFormat.ARGB32, false);
		}
		else
		{
			// Make the atlas readable so we can save it
            tex = NGUIEditorTools.ImportTexture(oldPath, true, false, false, bUIOptimization);
		}

		// Pack the sprites into this texture
		if (PackTextures(tex, sprites))
		{
			byte[] bytes = tex.EncodeToPNG();
			System.IO.File.WriteAllBytes(newPath, bytes);
			bytes = null;

			// Load the texture we just saved as a Texture2D
			AssetDatabase.SaveAssets();
			AssetDatabase.Refresh(ImportAssetOptions.ForceSynchronousImport);
			tex = NGUIEditorTools.ImportTexture(newPath, false, true, !atlas.premultipliedAlpha);

			// Update the atlas texture
			if (newTexture)
			{
				if (tex == null) LogUtil.LogError("Failed to load the created atlas saved as " + newPath);
				else atlas.spriteMaterial.mainTexture = tex;
				ReleaseSprites(sprites);
				
				AssetDatabase.SaveAssets();
				AssetDatabase.Refresh(ImportAssetOptions.ForceSynchronousImport);
			}

            if (tex != null && atlas.spriteMaterial.shader != null)
            {
                if (bUIOptimization)
                {
                    TextureProceseser.ProcessUITex(tex);
                }
                else
                {
                    if (atlas.spriteMaterial.shader.name == "Unlit/Transparent Colored Split")
                    {
                        atlas.spriteMaterial.shader = Shader.Find(NGUISettings.atlasPMA ? "Unlit/Premultiplied Colored" : "Unlit/Transparent Colored");
                        atlas.spriteMaterial.mainTexture = tex;

                        //Try delete alpha texture
                        string folderPath = StringUtils.ParseFolderPath(AssetDatabase.GetAssetPath(tex));
                        string alphaPath = folderPath + "/" + tex.name + "_a.png";
                        AssetDatabase.DeleteAsset(alphaPath);

                        AssetDatabase.SaveAssets();
                        AssetDatabase.Refresh(ImportAssetOptions.ForceSynchronousImport);
                    }
                }
            }


			return true;
		}
		else
		{
			if (!newTexture) NGUIEditorTools.ImportTexture(oldPath, false, true, !atlas.premultipliedAlpha);
			
			//LogUtil.LogError("Operation canceled: The selected sprites can't fit into the atlas.\n" +
			//	"Keep large sprites outside the atlas (use UITexture), and/or use multiple atlases instead.");
			
			EditorUtility.DisplayDialog("Operation Canceled", "The selected sprites can't fit into the atlas.\n" +
					"Keep large sprites outside the atlas (use UITexture), and/or use multiple atlases instead , ask Jubmo for help", "OK");
			return false;
		}
	}

```
* 修改NGUISettings.cs
  添加    uiOptimization
```
static public bool uiOptimization
    {
        get { return GetBool("NGUI UIOptimization", true); }
        set { SetBool("NGUI UIOptimization", value); }
    }
```

* 修改 NGUIEditorTools.cs
```
static public Texture2D ImportTexture(string path, bool forInput, bool force, bool alphaTransparency, bool bUIOptimization = false)
	{
		if (!string.IsNullOrEmpty(path))
		{
            if (forInput) { if (!MakeTextureReadable(path, force, bUIOptimization)) return null; }
            else if (!MakeTextureAnAtlas(path, force, alphaTransparency, bUIOptimization)) return null;
			//return AssetDatabase.LoadAssetAtPath(path, typeof(Texture2D)) as Texture2D;

			Texture2D tex = AssetDatabase.LoadAssetAtPath(path, typeof(Texture2D)) as Texture2D;
			AssetDatabase.Refresh(ImportAssetOptions.ForceSynchronousImport);
			return tex;
		}
		return null;
	}
```
```
 static public bool MakeTextureReadable(string path, bool force, bool bUIOptimization = false)
	{
		if (string.IsNullOrEmpty(path)) return false;
		TextureImporter ti = AssetImporter.GetAtPath(path) as TextureImporter;
		if (ti == null) return false;

		TextureImporterSettings settings = new TextureImporterSettings();
		ti.ReadTextureSettings(settings);

		if (force || !settings.readable || settings.npotScale != TextureImporterNPOTScale.None || settings.alphaIsTransparency)
		{
			settings.readable = true;
			if (NGUISettings.trueColorAtlas) settings.textureFormat = TextureImporterFormat.ARGB32;
			settings.npotScale = TextureImporterNPOTScale.None;
			settings.alphaIsTransparency = false;
			ti.SetTextureSettings(settings);
            //优化Andriod和iOS,防止有些图集被压缩了，原本的NGUI AtlasMaker无法重新打图集
            if (bUIOptimization)
            {
                ti.SetPlatformTextureSettings("iOS", ti.maxTextureSize, TextureImporterFormat.RGBA32, (int)TextureCompressionQuality.Normal);
                ti.SetPlatformTextureSettings("Android", ti.maxTextureSize, TextureImporterFormat.RGBA32, (int)TextureCompressionQuality.Normal);
                //AssetDatabase.WriteImportSettingsIfDirty(path);
            }
            
			AssetDatabase.ImportAsset(path, ImportAssetOptions.ForceUpdate | ImportAssetOptions.ForceSynchronousImport);
		}
		return true;
	}
```
```
static bool MakeTextureAnAtlas (string path, bool force, bool alphaTransparency, bool bUIOptimization = false)
	{
		if (string.IsNullOrEmpty(path)) return false;
		TextureImporter ti = AssetImporter.GetAtPath(path) as TextureImporter;
		if (ti == null) return false;

		TextureImporterSettings settings = new TextureImporterSettings();
		ti.ReadTextureSettings(settings);

		if (force ||
			settings.readable ||
			settings.maxTextureSize < 4096 ||
			settings.wrapMode != TextureWrapMode.Clamp ||
			settings.npotScale != TextureImporterNPOTScale.ToNearest)
		{
			settings.readable = false;
			settings.maxTextureSize = 4096;
			settings.wrapMode = TextureWrapMode.Clamp;
			settings.npotScale = TextureImporterNPOTScale.ToNearest;

			if (NGUISettings.trueColorAtlas)
			{
				settings.textureFormat = TextureImporterFormat.ARGB32;
				settings.filterMode = FilterMode.Trilinear;
			}

			settings.aniso = 4;
			settings.alphaIsTransparency = alphaTransparency;

            // [yqq] 2015-12-8 force disable mipmap for atlas texture
            settings.mipmapEnabled = false;

			ti.SetTextureSettings(settings);

            //优化Andriod和iOS,防止有些图集被压缩了，原本的NGUI AtlasMaker无法重新打图集
            if (bUIOptimization)
            {
                ti.SetPlatformTextureSettings("iOS", ti.maxTextureSize, TextureImporterFormat.RGBA32, (int)TextureCompressionQuality.Normal);
                ti.SetPlatformTextureSettings("Android", ti.maxTextureSize, TextureImporterFormat.RGBA32, (int)TextureCompressionQuality.Normal);
                //AssetDatabase.WriteImportSettingsIfDirty(path);
            }
            
			AssetDatabase.ImportAsset(path, ImportAssetOptions.ForceUpdate | ImportAssetOptions.ForceSynchronousImport);
		}
		return true;
	}
```

* Editor目录新建 TextureProceseser.cs
  * 添加如下方法


```
    public static void ChangeTextureImportSetting(Texture tex, bool readable, int size, TextureImporterFormat format = TextureImporterFormat.RGBA32, TextureImporterType importType = TextureImporterType.Advanced)
	{
		string path = AssetDatabase.GetAssetPath(tex);
		
		TextureImporter importer = (TextureImporter)TextureImporter.GetAtPath(path);
		importer.textureType = importType;
		importer.maxTextureSize = size;
		importer.textureFormat = format;

		importer.isReadable = true;
		AssetDatabase.ImportAsset(path, ImportAssetOptions.ForceUpdate);
	}

public static void ProcessUITex(Texture2D tex)
    {
        SliptAlphaChannelForlMobile(tex);
        //修改材质
        ModifyMaterial(tex);
        

        AssetDatabase.Refresh();

    }


private static void SliptAlphaChannelForlMobile(Texture2D tex)
    {
        // 将贴图的格式设置为RGBA32，获得相应的通道。
        TextureProceseser.ChangeTextureImportSetting(tex, true, Mathf.Max(tex.width, tex.height));

        TextureImporterFormat destTexFormat = TextureImporterFormat.AutomaticCompressed;
        TextureImporterFormat destAndriodTexFormat = (tex.width == tex.height && tex.width % 2 == 0) ? TextureImporterFormat.ETC_RGB4 : TextureImporterFormat.DXT5;
        TextureImporterFormat destIOSTexFormat = (tex.width == tex.height && tex.width % 2 == 0) ? TextureImporterFormat.PVRTC_RGB4 : TextureImporterFormat.RGB24;
        int maxTexSize = Mathf.Max(tex.width, tex.height);
        int width = tex.width;
        int height = tex.height;

        // 设置目标格式
        TextureImporterSettings settings = new TextureImporterSettings();
        settings.ApplyTextureType(TextureImporterType.Advanced, true);
        settings.mipmapEnabled = false;
        settings.npotScale = TextureImporterNPOTScale.ToNearest;
        settings.readable = false;
        settings.normalMap = false;
        settings.lightmap = false;
        settings.generateCubemap = TextureImporterGenerateCubemap.None;
        settings.wrapMode = TextureWrapMode.Clamp;
        settings.textureFormat = destTexFormat;
        settings.maxTextureSize = maxTexSize;
        settings.filterMode = FilterMode.Bilinear;
        settings.alphaIsTransparency = true;
        settings.aniso = 4;

        // 获取原始像素数据的引用。
        Color32[] pixels = tex.GetPixels32();
        int pixelCount = pixels.Length;
        Color32[] rawPixels = new Color32[pixelCount];
        for (int i = 0; i < pixelCount; ++i)
        {
            rawPixels[i] = new Color32(pixels[i].r, pixels[i].g, pixels[i].b, pixels[i].a);
        }
        string folderPath = StringUtils.ParseFolderPath(AssetDatabase.GetAssetPath(tex));
        string rgbPath = folderPath + "/" + tex.name + ".png";

        // 首先生成Alpha通道的贴图。
        Texture2D alpha = new Texture2D(tex.width, tex.height, TextureFormat.RGB24, false);
        Color32[] alphaPixels = new Color32[pixelCount];
        for (int i = 0; i < pixelCount; i++)
        {
            alphaPixels[i] = new Color32(rawPixels[i].a, rawPixels[i].a, rawPixels[i].a, rawPixels[i].a);
        }
        alpha.SetPixels32(alphaPixels);
        alpha.Apply(false);
        string fileName = tex.name + "_a";
        string alphaPath = folderPath + "/" + fileName + ".png";
        System.IO.File.WriteAllBytes(alphaPath, alpha.EncodeToPNG());

        /* 原图还是保留，不复写，至修改平台相关参数
        // 转换自己的格式，丢弃Alpha通道并保存。
        tex.Resize(width, height, TextureFormat.RGB24, false);
        tex.SetPixels32(rawPixels);
        tex.Apply(false);
        System.IO.File.WriteAllBytes(rgbPath, tex.EncodeToPNG());
        */
        AssetDatabase.Refresh();

        // 修改纹理的贴图设置
        TextureProceseser.ChangeTextureImportSetting(tex, settings);
        if (destTexFormat != destIOSTexFormat)
            TextureProceseser.ChangePlatformTextureImportSetting(tex, "iOS", maxTexSize, destIOSTexFormat);
        if (destTexFormat != destAndriodTexFormat)
            TextureProceseser.ChangePlatformTextureImportSetting(tex, "Android", maxTexSize, destAndriodTexFormat);

        alpha = AssetDatabase.LoadAssetAtPath(alphaPath, typeof(Texture2D)) as Texture2D;
        TextureProceseser.ChangeTextureImportSetting(alpha, settings);
        if (destTexFormat != destIOSTexFormat)
            TextureProceseser.ChangePlatformTextureImportSetting(alpha, "iOS", maxTexSize, destIOSTexFormat);
        if (destTexFormat != destAndriodTexFormat)
            TextureProceseser.ChangePlatformTextureImportSetting(alpha, "Android", maxTexSize, destAndriodTexFormat);

        // 刷新资源。
        AssetDatabase.Refresh();
    }

public static void ModifyMaterial(Texture2D tex)
    {

        string path = AssetDatabase.GetAssetPath(tex);
        int maxSize = Mathf.Max(tex.width, tex.height);
        if (tex == null || string.IsNullOrEmpty(path))
        {
            Debug.LogWarning("This Texture is null, " + path);
            return;
        }

        string foladerPath = StringUtils.ParseFolderPath(path);
        string matPath = path.Substring(0, path.Length - ".png".Length) + ".mat";
        Material mat = AssetDatabase.LoadAssetAtPath(matPath, typeof(Material)) as Material;
        //int start = path.LastIndexOf("/");
       // int end = path.LastIndexOf(".");
        //string name = path.Substring(start + 1, end - start - 1);
        string alphaPath = foladerPath + "/" + tex.name + "_a.png";
        if (mat != null)
        {
            if (mat.shader.name.CompareTo("Unlit/Transparent Colored Split") != 0)
            {
                mat.shader = Shader.Find("Unlit/Transparent Colored Split");
                mat.mainTexture = tex;
                //mat.SetTexture("_MainTex", tex);
                Texture2D alpha = AssetDatabase.LoadAssetAtPath(alphaPath, typeof(Texture2D)) as Texture2D;
                if (alpha != null)
                    mat.SetTexture("_MaskTex", alpha);
                else
                    Debug.LogError("ModifyMaterial : alpah is null path:" + alphaPath);


                AssetDatabase.SaveAssets();
                AssetDatabase.Refresh(ImportAssetOptions.ForceSynchronousImport);
            }
            
        }
    }
```





