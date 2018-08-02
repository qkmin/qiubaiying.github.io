---
layout:     post
title:      AndroidFramewok编译apk丢失drawable
subtitle:  
date:       2018-08-02
author:     qkmin
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android 
    - AndroidFramework
---
# 运行异常  
源码环境编译app运行报找不到资源文件，这就很郁闷了，明明install安装都完美运行啊。  
对比了下能运行的apk和源码编译报错的apk，发现drawable-hdpi缺失了？？？  
what？看来罪魁祸首是这个。   
  
# 解决  
- 在Android.mk文件中添加  
```
LOCAL_AAPT_FLAGS += -c ldpi -c mdpi -c hdpi
```
  
hdpi文件夹编译过后就有了