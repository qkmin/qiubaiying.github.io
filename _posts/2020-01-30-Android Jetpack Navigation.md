---
layout:     post
title:      Android Jetpack Navigation
subtitle:   Navigation
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
更确切的来说，Navigation是用来管理Fragment的切换，并且可以通过可视化的方式，看见App的交互流程。这完美的契合了Jake Wharton大神单Activity的建议。  

# Navigation使用  
##准备
Android studio3.2或者更高  
#添加依赖  
模块层的build.gradle文件需要添加：

```
ext.navigationVersion = "2.0.0"
dependencies {
    implementation "androidx.navigation:navigation-fragment-ktx:$rootProject.navigationVersion"
    implementation "androidx.navigation:navigation-ui-ktx:$rootProject.navigationVersion"
}
```
#创建navigation导航  

1. 创建基础目录：资源文件`res`目录下创建`navigation`目录 -> 右击`navigation`目录New一个`Navigation resource file`

2. 创建frament   

   ```
   <navigation
       ...
       android:id="@+id/login_navigation"
       app:startDestination="@id/welcome">
   
       <fragment
           android:id="@+id/login"
           android:name="com.joe.jetpackdemo.ui.fragment.login.LoginFragment"
           android:label="LoginFragment"
           tools:layout="@layout/fragment_login"
           />
   
       <fragment
           android:id="@+id/welcome"
           android:name="com.joe.jetpackdemo.ui.fragment.login.WelcomeFragment"
           android:label="LoginFragment"
           tools:layout="@layout/fragment_welcome">
           <action
               .../>
           <action
               .../>
       </fragment>
       </fragment>
   </navigation>
   
   ```

   app:startDestination="@id/welcome必须声明，表示默认的起始位置。

   在MainActivity的layout中

   ```
    <fragment
           android:id="@+id/my_nav_host_fragment"
           android:name="androidx.navigation.fragment.NavHostFragment"
           app:navGraph="@navigation/login_navigation"
           app:defaultNavHost="true"
           android:layout_width="match_parent"
           android:layout_height="match_parent"/>
   
   ```

   | 属性                        |                             解释                             |
   | :-------------------------- | :----------------------------------------------------------: |
   | `android:name`              | 值必须是`androidx.navigation.fragment.NavHostFragment`，声明这是一个`NavHostFragment` |
   | `app:navGraph`              | 存放的是第二步建好导航的资源文件，也就是确定了`Navigation Graph` |
   | `app:defaultNavHost="true"` |                    与系统的返回按钮相关联                    |

   跳转frament的方法

   ```
   
   Navigation.findNavController(Activity, @IdRes int viewId)
   
   Navigation.findNavController(View)
   
   ```

   


