---
layout:     post
title:      Android framework添加系统jar
subtitle:   
date:       2019-12-01
author:     qkmin
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - jar
    - Android framework
---

# Android framework添加系统jar

Android framewok需要使用自定义jar，我们可以把jar编译进系统，开机加载：

- 源码 framework/opt 目录新建文件夹 test，第一步生成的 test-1.0.0.1.jar 复制到此目录，并新建立 Android.mk，写法如下

```
LOCAL_PATH := $(call my-dir)
 
#test-1.0.0.1.jar
include $(CLEAR_VARS)
LOCAL_MODULE := test-1.0.0.1
LOCAL_MODULE_TAGS := optional
LOCAL_SRC_FILES := test-1.0.0.1.jar
LOCAL_MODULE_SUFFIX := $(COMMON_JAVA_PACKAGE_SUFFIX)
LOCAL_MODULE_CLASS := JAVA_LIBRARIES
include $(BUILD_PREBUILT
```

- build/target/product/core_minimal.mk 或 build/target/product/core_base.mk 文件 PRODUCT_PACKAGES += 和 PRODUCT_BOOT_JARS := 末尾分别增加 test-1.0.0.1 (上一步 LOCAL_MODULE := 的定义值)

```

# The order of PRODUCT_BOOT_JARS matters.
PRODUCT_BOOT_JARS := \
    core-libart \
    conscrypt \
    okhttp \
    core-junit \
    bouncycastle \
    ext \
    framework \
    telephony-common \
    voip-common \
    ims-common \
    apache-xml \
    org.apache.http.legacy.boot \
    test-1.0.0.1
```



- 修改 build/core/tasks/check_boot_jars/package_whitelist.txt，文件末尾增加 test-1.0.0.1.jar 包名，如：


```
###################################################
# test-1.0.0.1.jar
com\.king\.test\..*
```

- 然后编译


```
全编或者
make test-1.0.0.1
```

