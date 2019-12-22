---
layout: post
title:  "Android-SplashActivity-启屏制作(可直接在Unity主Activity前面显示)"
date:   2017-04-28 10:45:32 +0800
categories: [Android]
---

>版权声明：本文为Jumbo原创文章,采用[[知识共享 署名-非商业性使用-禁止演绎 4.0 国际 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)]，转载前请保证理解此协议
原文出处：[https://www.jianshu.com/p/b63b21e5e3e6](https://www.jianshu.com/p/b63b21e5e3e6)

## {{ page.title }}

使用场景：
    由于项目原因，我们需要在Unity启动前添加第三方Splash图片（或者动画），一边以游戏本身分离
故在Java层实现Android Splash

实现如下：
1、SplashActivity.java

```
package com.x.x;//包名

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.view.Window;

public class SplashActivity extends Activity {

    private final int SPLASH_DISPLAY_LENGHT = 2000;//启屏显示时间ms单位
    private Handler handler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getWindow().requestFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.activity_splash);

        handler = new Handler();
        // 延迟SPLASH_DISPLAY_LENGHT时间然后跳转到MainActivity
        handler.postDelayed(new Runnable() {

            @Override
            public void run() {
                Intent intent = new Intent(SplashActivity.this,
                        MainActivity.class);//游戏主Activity，Unity中是继承Unity Activity的那个Activity
                startActivity(intent);
                SplashActivity.this.finish();
            }
        }, SPLASH_DISPLAY_LENGHT);

    }
}
```

2、res\layout中添加activity_splash.xml 布局
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal" >

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:src="@drawable/splash_img_0" />

</LinearLayout>
```

3、res\drawable添加splash_img_0.png

4、修改AndroidManifest.xml 把SplashActivity改为游戏启动Activity，在Application标签内添加
```
<activity android:name=".SplashActivity"
                  android:label="你需要显示游戏的名称"
				  android:screenOrientation="landscape"
                  android:configChanges="mnc|keyboardHidden|screenSize|orientation|keyboard"
                  android:theme="@android:style/Theme.Black.NoTitleBar.Fullscreen" >

          <intent-filter>
          <action android:name="android.intent.action.MAIN" />

          <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>

        </activity>
```

5、如果之前你有设置其他Activity为主Activity，你需要在其所在的activity标签内注释以下，或者删除
```
<!--
 <intent-filter>
          <action android:name="android.intent.action.MAIN" />

          <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
-->
```
注意：
由于在不同安卓设备上，启屏Activity完成时，即 SplashActivity.this.finish()
测试：小米、华为Mate9会侧滑消失（测试左滑消失）
           LG Nexus 5x正常消失
诊断：可能是各家Rom定制，在底层改过activity消失动画，Google原生系统没有这个问题。
