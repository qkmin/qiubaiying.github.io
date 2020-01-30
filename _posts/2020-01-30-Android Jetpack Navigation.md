---
layout:     post
title:      Android Jetpack Navigation
subtitle:   apk上传蒲公英
date:       2020-01-30
author:     qkmin
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Jetpack 
    - Navigation
---
# Navigation简介  
![来自网络的一张图片](https://upload-images.jianshu.io/upload_images/9271486-af1f3f17e4392b27.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)  
Navigation是一个可简化Android导航的库和插件

##操作
###1使用命令上传 apk 到蒲公英：
```
curl -F "file=@/tmp/example.apk" -F "uKey=" -F "_api_key=" https://qiniu-storage.pgyer.com/apiv1/app/upload 
```


- **file后面添加本地生成apk地址**  
- **uKey后面添加应用User Key**
- **_api_key添加应用API Key**  
注意要写在引号里面    
插入构建
![shell](https://static.pgyer.com/image/view/admin_images/17a72d11aa544146ab0c127b0eb0882c)    

  

###2使用蒲公英插件  

-**[蒲公英链接](https://www.pgyer.com/doc/view/jenkins_plugin)**
