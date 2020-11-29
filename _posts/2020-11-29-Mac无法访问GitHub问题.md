---
layout:     post
title:      Github访问过慢或者失败问题
subtitle:   
date:       2020-11-29
author:     qkmin
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Github
    - Mac
---

# 1、访问过慢解决办法
```
sudo vim /etc/hosts

添加


# Github

151.101.185.194 github.global.ssl.fastly.net
192.30.253.112 github.com 
151.101.112.133 assets-cdn.github.com 
151.101.184.133 assets-cdn.github.com 
185.199.108.153 documentcloud.github.com 
192.30.253.118 gist.github.com
185.199.108.153 help.github.com 
192.30.253.120 nodeload.github.com 
151.101.112.133 raw.github.com 
23.21.63.56 status.github.com 
192.30.253.1668 training.github.com 
192.30.253.112 www.github.com 
151.101.13.194 github.global.ssl.fastly.net 
151.101.12.133 avatars0.githubusercontent.com 
151.101.112.133 avatars1.githubusercontent.com



刷新dns

dscacheutil -flushcache    
```

#2、无法访问问题
```
通过查看 https://github.com.ipaddress.com ，查看github.com地址为140.82.114.

sudo vim /etc/hosts

添加

140.82.114.4 github.com 
```
可以正常访问了！