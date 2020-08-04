---
title: Sdp
date: 2020-07-30 19:37:45
tags:
---

# sdp

## sdp 中的流媒体

### video

#### H264

h264 有四种级别的画质，分别是baseline、extended、main 和high。
* baseline 基本画质，支持I/P 帧
* extended 进阶画质，支持I/P/B/SP/SI 帧(后两个没听说过) 
* main     主流画质，支持I/P/B 帧
* high     高级画质

webrtc 在sdp 交换中有对视频描述如下：
```shell
a=fmtp:102 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42001f
a=fmtp:127 level-asymmetry-allowed=1;packetization-mode=0;profile-level-id=42001f
a=fmtp:125 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42e01f
a=fmtp:108 level-asymmetry-allowed=1;packetization-mode=0;profile-level-id=42e01f
a=fmtp:124 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=4d0032
a=fmtp:123 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=640032
```
其中profile-level-id=xxxxxx 就是H264 profile和level 的组合，xxxxxx 可以分为三部分，每部分为两个十六进制的数字，从左到右依次为profile_idc(表示画质)、profile_iop和level_idc(码率和分辨率限制)。一般关注第一个和第三个。

https://blog.csdn.net/liang12360640/article/details/52096499
## sdp 中的setup 

a=setup 主要是表示dtls 协商中角色的问题，谁是客户端，谁是服务器。

```shell
# 既可以是服务端 也可以是客户端
a=setup:actpass
# 客户端
a=setup:active
# 服务端
a=setup:passive
```

https://blog.csdn.net/glw0223/article/details/91871718
https://blog.csdn.net/m0_37263637/article/details/96355737

## sdp 中的 trickle

```shell
#通知对端支持trickle，即sdp里面描述媒体信息和ice候选项的信息可以分开传输
a=ice-options:trickle
```

https://www.jianshu.com/p/61e3c9e13456
