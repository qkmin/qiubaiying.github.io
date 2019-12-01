---
s layout:     post
title:      Android framework添加资源
subtitle:   
date:       2019-11-22
author:     qkmin
header-img: img/post-bg-blue.jpg
catalog: true
tags:
    - res
    - Android framework
---

# Android 8.0添加系统资源



##一.增加png类型的图片资源　　

1.将appupdate模块所有用到的png格式图片拷贝到framework/base/core/res/res/drawable-mdpi里。但是要确保没有与原生的没有重名文件。　　

2.在framework/base/core/res/res/values/symbols.xml文件里增加对这些图片的声明。

3.framework/base/core/res/res/ 下mm编译

4.make framework

##二.增加string资源　　

1.将appupdate模块定义的string.xml里面的所以string拷贝到framework/base/core/res/res/values/string.xml里。但是确保没有重名的。　　

2.在framework/base/core/res/res/values/public.xml文件里增加对这些string的声明。(例：此id要保证唯一，以string类型的最后一个id为基数增加)　　

3.framework/base/core/res/res/ 下mm编译　　

4.make framework　

##三.增加layout资源　　

1.将appupdate模块的layout文件里定义的5个xml文件拷贝到在framework/base/core/res/res/layout里。但是要确保没有重名文件被覆盖。　　

2.在framework/base/core/res/res/values/public.xml文件里增加对这些layout的声明。(例：此id要保证唯一，以layout类型的最后一个id为基数增加)　　

3.framework/base/core/res/res/ 下mm编译　　

4.make framework　

说明：若layout中包含xml,直接把xml拷贝到framework相应目录下。

例如

(1)button的selector。将appupdate模块的drawable/common_btn_selector.xml文件拷贝到framework/base/core/res/res/drawable里，确保没有重名文件。　　

(2)将appupdate模块的anim/loading.xml文件拷贝到framework/base/core/res/res/anim里，确保没有重名文件。　　

##四.增加style资源　　

1.将appupdate模块的style文件里定义的所有style拷贝到framework/base/core/res/res/values/style.xml里。确保没有覆盖原生的style.　　

2.在framework/base/core/res/res/values/public.xml文件里增加对这些style的声明。　　3.framework/base/core/res/res/ 下mm编译　　

4.make framework　

##五.增加color资源　　

1.将appupdate模块的style文件里定义的所有style拷贝到framework/base/core/res/res/values/color.xml里。确保没有覆盖原生的color.　　
2.在framework/base/core/res/res/values/public.xml文件里增加对这些color的声明。　　3.framework/base/core/res/res/ 下mm编译　　
4.make framework　

##六.增加资源id　　

1.在framework/base/core/res/res/values/ids.xml里定义你jar中所用的id(R.id)*)。确保没有覆盖原生的.　　
2.在framework/base/core/res/res/values/public.xml文件里增加对这些id的声明。　　3.framework/base/core/res/res/ 下mm编译　　
4.make framework

## 总结

- 源文件路径: /frameworks/base/core/res/res

- 编译后路径: /out/target/product/项目名称/system/framework/framework-res.apk

- R.Java文件: /out/target/common/R/com/android/internal/R.Java

- 资源的添加:
  如在framework-res中添加一个共有字符串，需修改以下文件：
      frameworks/base/core/res/res/values/public.xml
      frameworks/base/core/res/res/values/strings.xml

  如在framework-res中添加一个私有字符串，需修改以下文件：
      frameworks/base/core/res/res/values/symbols.xml
      frameworks/base/core/res/res/values/strings.xml

- 添加完成后进行 mmm framework/base/core/res 编译,然后检查是否添加到R.java文件中.
  添加成功后 在代码中使用 com.android.internal.R.string.xxx 来引用.
- 在对系统新增了一些资源进行源码编译时会遇到 com.android.internal.R.XX can not find 的问题,可使用（make update-api）来更新api,
  ./frameworks/base/api/current.txt 会被重新生成.

- public.xml 与 symbols.xml

- public.xml中声明的是公共资源,所有应用程序都可以调用,symbols.xml中声明的是非公共资源,仅供系统内部使用,不对app开放.
  public.xml 中字段格式为 <public type="attr" name="networkSecurityConfig" id="0x01010527" />
  symbols.xml中字段格式为<java-symbol type="string" name="use_times"/>
  symbols.xml是在4.2后,将系统私有的资源分离成了单独的文件.
  若将私有的声明添加在了public.xml文件中,则编译时会报错,可采用 make framework 2>&1 | sed -n -f MakeJavaSymbols.sed | sort -u ,
  此命令将会列出所有新增的私有资源，并将它们拷贝到symbols.xml中.
