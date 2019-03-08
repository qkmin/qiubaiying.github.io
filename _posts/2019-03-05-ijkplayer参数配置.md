---
layout:     post
title:      ijkplayer编译
subtitle:   
date:       2019-03-05
author:     qkmin
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - ijkplayer编译
    - Android播放器
---
# 引言
公司的直播项目要求兼容rtsp协议,调用原生的mediaplayer播放不了。网上开源的播放器有：
- ExoPlayer 

- Vitamio

- ijkplayer

最终选用ijkplayer，基于FFmpeg的轻量级Android/iOS视频播放器，最主要的百度资料很多，坑少。
# 编译
ijkplayer的 github项目地址：https://github.com/bilibili/ijkplayer
Before Build
	安装 git, yasm 配置NDK export ANDROID_NDK="/android-ndk-r12b"
```
	cd config
	rm module.sh
	ln -s module-default.sh module.sh
	cd android/contrib
```
Build Android
```
	git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
	cd ijkplayer-android
	./init-android.sh

	cd android/contrib
	./compile-ffmpeg.sh clean
    ./compile-ffmpeg.sh all

    cd ..
    ./compile-ijk.sh all
```
编译成功后我们可以导入demo来查看用法

