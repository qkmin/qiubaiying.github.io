---
layout:     post
title:      system app 静默安装
subtitle:   静默安装
date:       2018-10-09
author:     qkmin
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - 静默安装
    - Android Framework
---
# 前言

>最近用到静默安装，在这里记录一下。



# 常见安装

> 调用系统PackageInstaller安装，7.0以下版本

```

  Intent intent = new Intent();
  intent.setAction(Intent.ACTION_VIEW);
  intent.setDataAndType(Uri.fromFile(file),"application/vnd.android.package-archive");
  intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
  context.startActivity(intent);

```

> Root后执行shell命令安装

 ```
"pm install -r  /sdcard/new.apk"
"cp /sdcard/new.apk /data/app"
 ```
# System app静默安装
>使用PackageManager.installPackage(),需要系统权限
>

```
 public abstract void installPackage(
            Uri packageURI, IPackageInstallObserver observer, int flags,
            String installerPackageName);
```

> packageURI是apk的路径uri，observer是安装完成后的回调，flags有以下标准，1是普通，2是覆盖，4是测试的apk;installerPackageName 是安装的包名

     public static final int INSTALL_FORWARD_LOCK = 0x00000001;
     public static final int INSTALL_REPLACE_EXISTING = 0x00000002;
     public static final int INSTALL_ALLOW_TEST = 0x00000004;

> IPackageInstallObserver可以写个类继承

```
static class PackageInstallObserver extends IPackageInstallObserver.Stub {
		public void packageInstalled(String packageName, int returnCode) {
		}
	}
```

最后别忘记加权限

`<uses-permission android:name="android.permission.INSTALL_PACKAGES" />`