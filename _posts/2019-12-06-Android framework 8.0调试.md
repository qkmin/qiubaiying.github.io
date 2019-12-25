---
layout:     post
title:      Android framework8.0调试
subtitle:   
date:       2019-12-06
author:     qkmin
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android 8.0 mstar 8386
    - Android framework
---

# 查看进程

ps -A  
#framework jar生成
修改  device/mstar/xx/BoardConfigCommon.mk  
WITH_DEXPREOPT =false
mm就可以生成framework.jar包，调试起来就方便  
#显示BootClass

$BOOTCLASSPATH  
#应用调用so库报错

添加so库到package_whitelist.txt  
#关闭内核打印

echo 1       4       1      7 > /proc/sys/kernel/printk  
#恢复出厂设置

```
	private void intoRecovery(){
		IntentresetIntent;
		if(android.os.Build.VERSION.RELEASE.equals("8.0.0")){
			resetIntent=newIntent("android.intent.action.FACTORY_RESET");
			resetIntent.setPackage("android");
			resetIntent.setFlags(Intent.FLAG_RECEIVER_FOREGROUND);
			resetIntent.putExtra(Intent.EXTRA_REASON,"ResetConfirmFragment");
			if(activity.getIntent().getBooleanExtra("shutdown",false)){
				resetIntent.putExtra("shutdown",true);
			}
			}else{
				resetIntent=newIntent("android.intent.action.MASTER_CLEAR");
				resetIntent.addFlags(Intent.FLAG_RECEIVER_FOREGROUND);
				resetIntent.putExtra("from","restorefactory");
			}
			context.sendBroadcast(resetIntent);
	}

```

#修改设备的mac地址
```
Linux ：
$ sudo ifconfig eth0 down
$ sudo ifconfig eth0 hw ether 80:00:00:00:00:01
$ sudo ifconfig eth0 up
Android：
Ethernet 
# busybox ifconfig eth0 down
# busybox ifconfig eth0 hw ether 80:00:00:00:00:01
# busybox ifconfig eth0 up
or
Wlan
# busybox ifconfig wlan0 down
# busybox ifconfig wlan0 hw ether 80:00:00:00:00:01
# busybox ifconfig wlan0 up

```

