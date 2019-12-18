---
layout:     post
title:      Android壁纸
subtitle:  
date:       2019-12-15
author:     qkmin
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android 

---

#设置壁纸方法
```
WallpaperManager wallpaperManager = (WallpaperManager) mActivity.getSystemService(
                        Context.WALLPAPER_SERVICE);
wallpaperManager.setBitmap();
wallpaperManager.setResource();
wallpaperManager.setStream();
```

#设置壁纸需要的权限

```
<uses-permission android:name="android.permission.SET_WALLPAPER"/>
```

#获取壁纸方法

```
WallpaperManager wallpaperManager = (WallpaperManager) mActivity.getSystemService(
                        Context.WALLPAPER_SERVICE);
wallpaperManager.getDrawable()
```



