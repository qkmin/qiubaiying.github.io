---
layout:     post
title:      ijkplayer设置
subtitle:   
date:       2019-03-28
author:     qkmin
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - ijkplayer设置
    - Android播放器
---

ijkplayer 硬解码
```
在IjkVideoView类createPlayer（）
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "mediacodec", 1);
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "mediacodec-auto-rotate", 1);
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "mediacodec-handle-resolution-change", 1);
```
我们在demo提供的Settings类修改配置文件也可以
```
public boolean getUsingMediaCodec() {
        String key = mAppContext.getString(R.string.pref_key_using_media_codec);
        return mSharedPreferences.getBoolean(key, true);
    }

public boolean getUsingMediaCodecAutoRotate() {
  	String key = mAppContext.getString(R.string.pref_key_using_media_codec_auto_rotate);
        return mSharedPreferences.getBoolean(key, true);
    }

public boolean getMediaCodecHandleResolutionChange() {
   		String key = mAppContext.getString(R.string.pref_key_media_codec_handle_resolution_change);
        return mSharedPreferences.getBoolean(key, true);
    }
```
rtsp协议支持
```

ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "framedrop", 1L);
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "start-on-prepared", 0);
// 设置最长分析时长
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_FORMAT, "analyzemaxduration", 100L);
// 通过立即清理数据包来减少等待时长
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_FORMAT, "flush_packets", 1L);	ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_FORMAT, "http-detect-range-support", 0);				
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_CODEC, "skip_loop_filter", 48);//48 to 0			
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "probesize", "524288");  //for first display fast, but maybe can not play succ
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_FORMAT, "rtsp_transport", "tcp");

```
H265硬解码

```
h265硬解码失败后自动转成软解
error:ffpipenode_create_video_decoder_from_android_mediacodec: MediaCodec/HEVC is disabled. codec_id:174

//打开h265硬解
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "mediacodec-hevc", 1);
```
HTTP error 404 Not Found

```
// 清空DNS,有时因为在APP里面要播放多种类型的视频(如:MP4,直播,直播平台保存的视频,和其他http视频), 有时会造成因为DNS的问题而报10000问题的
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_FORMAT, "dns_cache_clear", 1);
```
缓冲设置

```
ijkMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "packet-buffering", 1L); // 1 打开缓冲(会回调info)， 0 不缓冲会卡顿（不会回调info)
```
ijkplayer默认抢占焦点
```
我的ijkplayer主要使用在Tv，默认会请求焦点，改成false
		setFocusable(false);
		setFocusableInTouchMode(false);
//		requestFocus();
```


混淆

```
-keep class tv.danmaku.ijk.media.player.** {*;}
-keep class tv.danmaku.ijk.media.player.IjkMediaPlayer{*;}
-keep class tv.danmaku.ijk.media.player.ffmpeg.FFmpegApi{*;}
```



未完待续...