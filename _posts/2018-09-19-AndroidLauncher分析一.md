---
layout:     post
title:      Launcher3分析<一>
subtitle:   源码分析
date:       2018-09-19
author:     BY
header-img: img/post-bg-android.png
catalog: true
tags:
    - Android
    - Launcher
---
# 前言  
最近需要实现一个自己的Launcher，就借机学习下原生的Launcher源码。  
## 多个Launcher
源码里有Launcher，Launcher2，Launcher3。那它们有什么区别呢。  
launcher不支持桌面小工具动画效果，launcher2添加了动画效果和3D初步效果支持。
   
Android 4.4 (KK)开始Launcher默认使用Launcher3，Launcher3较Launcher2 UI 有部分调整，主要包括:
  
*  状态栏透明，App List 透Wallpaper；
* 增加overview模式，可以调整workspace上页面的前后顺序；      
* 动态管理屏幕数量；       
* widget列表与app list分开显示；
*      默认不支持预置appwidget，需要用户指定权限；       
*       提供类似小米只有workspace的桌面机制 ；       
*       Wallpaper 的代码全部搬移到Launcher包；
*      类似Cling等细节的小变化；  
那我们就直接研究最新的Launcher3
#Launcher3  
## Launcher3介绍
Launcher是开机启动的第一个应用程序，用来展示应用列表和快捷方式、小部件等。
## Launcher3源码解析  
### AndroidManifest.xml    
一些不常用的标签
![123.png](https://upload-images.jianshu.io/upload_images/2607554-ee9755ecd74028f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
### java