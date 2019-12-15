---
layout:     post
title:      Android framework 壁纸
subtitle:   
date:       2019-12-15
author:     qkmin
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 壁纸
	- Android
---

#设置壁纸方法
```
 WallpaperManager.getInstance(context).setBitmap();
 WallpaperManager.getInstance(context).setResource();
 WallpaperManager.getInstance(context).setStream();
```

#设置壁纸需要的权限

```
<uses-permission android:name="android.permission.SET_WALLPAPER"/>
```

#获取壁纸方法

```
WallpaperManager.getInstance(this).getDrawable()
```



