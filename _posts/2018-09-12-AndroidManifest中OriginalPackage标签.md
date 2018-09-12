---
layout:     post
title:      AndroidManifest中Original Package标签
subtitle:   Original Package
date:       2018-09-12
author:     BY
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - Android
    - AndroidFramework
---



# 前言  
```
<original-package android:name="com.android.packageinstaller" />
```    
最近看到系统app的AndroidManifest中有个标签，不是很懂什么含义。 
  
# original-package 作用  
通过源码搜索original-package，在源码中找到两处。   
```
  frameworks/base/core/java/android/content/pm/PackageParser.java:1274   
```
```
  frameworks/base/services/java/com/android/server/BackupManagerService.java:3918:
```   
我们先来看PackageParser.java类     

		else if (tagName.equals("original-package")) {
		                sa = res.obtainAttributes(attrs,
		                        com.android.internal.R.styleable.AndroidManifestOriginalPackage);

                String orig =sa.getNonConfigurationString(
                        com.android.internal.R.styleable.AndroidManifestOriginalPackage_name, 0);
                if (!pkg.packageName.equals(orig)) {
                    if (pkg.mOriginalPackages == null) {
                        pkg.mOriginalPackages = new ArrayList<String>();
                        pkg.mRealPackage = pkg.packageName;
                    }
                    pkg.mOriginalPackages.add(orig);
                }

                sa.recycle();

                XmlUtils.skipCurrentTag(parser);
                
            } 
                String orig =sa.getNonConfigurationString(
                        com.android.internal.R.styleable.AndroidManifestOriginalPackage_name, 0);
                if (!pkg.packageName.equals(orig)) {
                    if (pkg.mOriginalPackages == null) {
                        pkg.mOriginalPackages = new ArrayList<String>();
                        pkg.mRealPackage = pkg.packageName;
                    }
                    pkg.mOriginalPackages.add(orig);
                }

                sa.recycle();

                XmlUtils.skipCurrentTag(parser);
                
            } 
    
AndroidManifestOriginalPackage 的定义，在 attrs_manifest.xml 文件中：   
    <!-- Private tag to declare the original package name that this package is
         based on.  Only used for packages installed in the system image.  If
         given, and different than the actual package name, and the given
         original package was previously installed on the device but the new
         one was not, then the data for the old one will be renamed to be
         for the new package.

         <p>This appears as a child tag of the root
         {@link #AndroidManifest manifest} tag. -->
    <declare-styleable name="AndroidManifestOriginalPackage" parent="AndroidManifest">
        <attr name="name" />
    </declare-styleable>
google翻译一下   
```
用于声明此程序包所在的原始程序包名称的私有标记基于。 仅用于系统映像中安装的软件包。 如果给定的，不同于实际的包名和给定的原始软件包以前安装在设备上但是新的一个不是，那么旧的数据将被重命名为对于新应用。
```
      
官方的解释很清楚了：之前安装的应用是系统应用，并且包名不同，之前应用的数据就会以新安装应用的名字保留下来。   


上面 parsePackage() 方法中的 mOriginalPackages，查源码可知对 mOriginalPackages 的其余处理只在 PackageManagerServerice 中了。我们来看PMS的scanPackageLI方法
   
 
		if (pkg.mOriginalPackages != null) {
                // This package may need to be renamed to a previously
                // installed name.  Let's check on that...
                final String renamed = mSettings.mRenamedPackages.get(pkg.mRealPackage);
                if (pkg.mOriginalPackages.contains(renamed)) {
                    // This package had originally been installed as the
                    // original name, and we have already taken care of
                    // transitioning to the new one.  Just update the new
                    // one to continue using the old name.
                    realName = pkg.mRealPackage;
                    if (!pkg.packageName.equals(renamed)) {
                        // Callers into this function may have already taken
                        // care of renaming the package; only do it here if
                        // it is not already done.
                        pkg.setPackageName(renamed);
                    }
                    
                } else {
                    for (int i=pkg.mOriginalPackages.size()-1; i>=0; i--) {
                        if ((origPackage = mSettings.peekPackageLPr(
                                pkg.mOriginalPackages.get(i))) != null) {
                            // We do have the package already installed under its
                            // original name...  should we use it?
                            if (!verifyPackageUpdateLPr(origPackage, pkg)) {
                                // New package is not compatible with original.
                                origPackage = null;
                                continue;
                            } else if (origPackage.sharedUser != null) {
                                // Make sure uid is compatible between packages.
                                if (!origPackage.sharedUser.name.equals(pkg.mSharedUserId)) {
                                    Slog.w(TAG, "Unable to migrate data from " + origPackage.name
                                            + " to " + pkg.packageName + ": old uid "
                                            + origPackage.sharedUser.name
                                            + " differs from " + pkg.mSharedUserId);
                                    origPackage = null;
                                    continue;
                                }
                            } else {
                                if (DEBUG_UPGRADE) Log.v(TAG, "Renaming new package "
                                        + pkg.packageName + " to old name " + origPackage.name);
                            }
                            break;
                        }
                    }
                }
            }



   
我们看到有英文提示，当mOriginalPackages!=null并与原来名字一样的时候 ,会赋值realName = pkg.mRealPackage;  
然后调用Settings.java(frameworks\base\services\java\com\android\server\pm)的getPackageLPw（），看下这个方法

			if (origPackage != null) {
                // We are consuming the data from an existing package.
                p = new PackageSetting(origPackage.name, name, codePath, resourcePath,
                        nativeLibraryPathString, vc, pkgFlags);
                if (PackageManagerService.DEBUG_UPGRADE) Log.v(PackageManagerService.TAG, "Package "
                        + name + " is adopting original package " + origPackage.name);
                // Note that we will retain the new package's signature so
                // that we can keep its data.
                PackageSignatures s = p.signatures;
                p.copyFrom(origPackage);
                p.signatures = s;
                p.sharedUser = origPackage.sharedUser;
                p.appId = origPackage.appId;
                p.origPackage = origPackage;
                mRenamedPackages.put(name, origPackage.name);
                name = origPackage.name;
                // Update new package state.
                p.setTimeStamp(codePath.lastModified());
            } else {
                p = new PackageSetting(name, realName, codePath, resourcePath,
                        nativeLibraryPathString, vc, pkgFlags);
                p.setTimeStamp(codePath.lastModified());
                p.sharedUser = sharedUser;
                // If this is not a system app, it starts out stopped.
                if ((pkgFlags&ApplicationInfo.FLAG_SYSTEM) == 0) {
                    if (DEBUG_STOPPED) {
                        RuntimeException e = new RuntimeException("here");
                        e.fillInStackTrace();
                        Slog.i(PackageManagerService.TAG, "Stopping package " + name, e);
                    }
                    List<UserInfo> users = getAllUsers();
                    if (users != null && allowInstall) {
                        for (UserInfo user : users) {
                            // By default we consider this app to be installed
                            // for the user if no user has been specified (which
                            // means to leave it at its original value, and the
                            // original default value is true), or we are being
                            // asked to install for all users, or this is the
                            // user we are installing for.
                            final boolean installed = installUser == null
                                    || installUser.getIdentifier() == UserHandle.USER_ALL
                                    || installUser.getIdentifier() == user.id;
                            p.setUserState(user.id, COMPONENT_ENABLED_STATE_DEFAULT,
                                    installed,
                                    true, // stopped,
                                    true, // notLaunched
                                    false, // blocked
                                    null, null, null);
                            writePackageRestrictionsLPr(user.id);
                        }
                    }
                }   



使用现有应用的数据，实现数据共享。   
我们总算找到这个标签的作用了。恭喜！