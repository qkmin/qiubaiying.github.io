---
layout:     post
title:      AndroidTvLauncher
subtitle:  
date:       2018-08-24
author:     qkmin
header-img: img/post-bg-miui6.jpg
catalog: true
tags:
    - Android 
    - AndroidFramework
---
# 需求   
![效果图](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1535114917308&di=c285611363c545cf061b40d76cb9efd9&imgtype=0&src=http%3A%2F%2Fimg.zcool.cn%2Fcommunity%2F01bc4e5544d8a40000019ae91367a0.jpg%402o.jpg)
最近有个需求做Tv的Launcher，要求要滑动，还有导航栏，第一次做，遇到很多坑，一起来看下吧！
  
# 实现  
* TabLayout  
第一想到的是用TabLayout实现，但实际用途中发现焦点会丢失。  可以参考这个[Demo](https://github.com/zhousuqiang/TvRecyclerView)  ,这个项目就是用TabLayout+自定义RecyclerView，有兴趣的可学习。
* ViewPager+LinearLayout  
使用ViewPager滑动，LinearLayout 的addView 添加Tab。TabLayout也是用这个实现的，只是封装好了。我们用这种实现就要自己处理keyevent
，比如viewpager的页面到tab栏，tab栏在到viewpager页面，等等。最主要的问题就是焦点的处理，因为不能触屏控制只能通过遥控器。代码以后补上！！！

