---
layout:     post
title:      Launcher启动过程
subtitle:   Android 8.0
date:       2019-12-27
author:     qkmin
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android 
    - AndroidFramework
---

# launcher启动  
Android开机过程中，会把各种系统服务拉起，并且调用其systemReady()函数，其中最关键的ActivityManagerService拉起后，systemReady()中调用了一个函数`startHomeActivityLocked()`，

```
public void systemReady(final Runnable goingCallback, TimingsTraceLog traceLog) {
...
    startHomeActivityLocked(currentUserId, "systemReady");
...
}
```

```

boolean startHomeActivityLocked(int userId, String reason) {
        if (mFactoryTest == FactoryTest.FACTORY_TEST_LOW_LEVEL
                && mTopAction == null) {
            // We are running in factory test mode, but unable to find
            // the factory test app, so just sit around displaying the
            // error message and don't try to start anything.
            return false;
        }
        Intent intent = getHomeIntent();
        ActivityInfo aInfo = resolveActivityInfo(intent, STOCK_PM_FLAGS, userId);
        if (aInfo != null) {
            intent.setComponent(new ComponentName(aInfo.applicationInfo.packageName, aInfo.name));
            // Don't do this if the home app is currently being
            // instrumented.
            aInfo = new ActivityInfo(aInfo);
            aInfo.applicationInfo = getAppInfoForUser(aInfo.applicationInfo, userId);
            ProcessRecord app = getProcessRecordLocked(aInfo.processName,
                    aInfo.applicationInfo.uid, true);
            if (app == null || app.instr == null) {
                intent.setFlags(intent.getFlags() | FLAG_ACTIVITY_NEW_TASK);
                final int resolvedUserId = UserHandle.getUserId(aInfo.applicationInfo.uid);
                // For ANR debugging to verify if the user activity is the one that actually
                // launched.
                final String myReason = reason + ":" + userId + ":" + resolvedUserId;
                mActivityStartController.startHomeActivity(intent, aInfo, myReason);
            }
        } else {
            Slog.wtf(TAG, "No home screen found for " + intent, new Throwable());
        }

        return true;
    }

```

getHomeIntent函数

```
Intent getHomeIntent() {
        Intent intent = new Intent(mTopAction, mTopData != null ? Uri.parse(mTopData) : null);
        intent.setComponent(mTopComponent);
        intent.addFlags(Intent.FLAG_DEBUG_TRIAGED_MISSING);
        if (mFactoryTest != FactoryTest.FACTORY_TEST_LOW_LEVEL) {
            intent.addCategory(Intent.CATEGORY_HOME);
        }
        return intent;
    }
```

这里会首先构造一个带有HOME category的Intent，用此intent来从PackageManagerService查询对应的ActivityInfo。Settings、Launcher、setupwizard（开机向导）中有声明，那么会启动哪个呢？看android:priority（[-1000, 1000]），Settings的FallbackHome 优先级高，并且Settings是在Manifest中声明了android:directBootAware="true"，因此刚开机时，其实只有FallbackHome这个页面会被检索到。

FallbackHome是个没有界面的activity，功能就是一直查找launcher，找到launcher后，启动launcher干掉自己。

```
private void maybeFinish() {
        if (getSystemService(UserManager.class).isUserUnlocked()) {
            final Intent homeIntent = new Intent(Intent.ACTION_MAIN)
                    .addCategory(Intent.CATEGORY_HOME);
            final ResolveInfo homeInfo = getPackageManager().resolveActivity(homeIntent, 0);
            if (Objects.equals(getPackageName(), homeInfo.activityInfo.packageName)) {
                if (UserManager.isSplitSystemUser()
                        && UserHandle.myUserId() == UserHandle.USER_SYSTEM) {
                    // This avoids the situation where the system user has no home activity after
                    // SUW and this activity continues to throw out warnings. See b/28870689.
                    return;
                }
                Log.d(TAG, "User unlocked but no home; let's hope someone enables one soon?");
                mHandler.sendEmptyMessageDelayed(0, 500);
            } else {
                Log.d(TAG, "User unlocked and real home found; let's go!");
                getSystemService(PowerManager.class).userActivity(
                        SystemClock.uptimeMillis(), false);
                finish();
            }
        }
    }
```

finish之后系统会再次调用到AMS的`startHomeActivityLocked`，这时候就已经可以查询到两个声明CATEGORY_HOME的activity了。但此时依然不会有应用选择对话框。这是因为FallbackHome在manifest中声明了自己的优先级为-1000，PackageManagerService里面对这样的情况是做了处理的。  

#开机向导启动  
启动FallbackHome后，第一次会启动开机向导，因为开机向导的优先级比launcher高

进入Google开机向导后，首先判断系统是否需要进入开机向导，通过以下三个参数的值：



```

Settings$Global.DEVICE_PROVISIONED

Settings$Secure.USER_SETUP_COMPLETE

System property: ro.setupwizard.mode (默认配置为OPTIONAL)
```

`DEVICE_PROVISIONED`用来标识是否走完开机向导
 `USER_SETUP_COMPLETE`与`ro.setupwizard.mode`是为多用户系统服务的，所知不多，暂不深入分析。

`DEVICE_PROVISIONED`为0，表示需要正常进入开机向导流程。

#开机向导为什么只会启动一次  

退出开机向导后会把HOME category禁用