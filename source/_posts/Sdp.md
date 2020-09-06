---
title: Sdp
date: 2020-07-30 19:37:45
tags:
    - webrtc
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

## sdp 中的 音视频描述
```shell
m=video 9 UDP/TLS/RTP/SAVPF 96 97 98 99 100 101 102 122 127 121 125 107 108 109 124 120 123 119 114 115 116
```

AVP     ==> audio video profile 不会启用rtcp 反馈，也不会根据rtcp反馈动态调整码率
AVPF    ==> audio video profile feedback 
SAVPF   ==> safe audio video profile feedback

https://segmentfault.com/a/1190000020794391


## sdp 的格式

webrtc 中，unified plan、plan B、plan A 是SDP 中多路媒体略的协商方式，在72 版本中，Chrome 默认使用unified plan 替换了Plan B。
在浏览器端，打开chrome://webrtc-internals 可以查看到PeerConnection 使用的sdp 协商方式。
```shell
https://webrtc.github.io/samples/src/content/peerconnection/pc1/, { iceServers: [], iceTransportPolicy: all, bundlePolicy: balanced, rtcpMuxPolicy: require, iceCandidatePoolSize: 0, sdpSemantics: "unified-plan" },
```

* plan B : sdp 中，一个m 行描述多路media stream 以，msid 作为区分
```shell
a=group:BUNDLE audio
a=msid-semantic: WMS stream-id-2 stream-id-1
m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126
...
a=mid:audio
...
a=rtpmap:103 ISAC/16000
...
a=ssrc:10 cname:cname
a=ssrc:10 msid:stream-id-1 track-id-1
a=ssrc:10 mslabel:stream-id-1
a=ssrc:10 label:track-id-1
a=ssrc:11 cname:cname
a=ssrc:11 msid:stream-id-2 track-id-2
a=ssrc:11 mslabel:stream-id-2
a=ssrc:11 label:track-id-2
```
* unified plan: 一个m 行对弈一个media stream
```shell
a=group:BUNDLE 0 1
a=msid-semantic: WMS
m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126
...
a=mid:0
...
a=sendrecv
a=msid:- <track-id-1>
...
a=rtpmap:103 ISAC/16000
...
a=ssrc:10 cname:cname
a=ssrc:10 msid: track-id-1
a=ssrc:10 mslabel:
a=ssrc:10 label:track-id-1
m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126
...
a=mid:1
...
a=sendrecv
a=msid:- track-id-2
...
a=rtpmap:103 ISAC/16000
...
a=ssrc:11 cname:cname
a=ssrc:11 msid: track-id-2
a=ssrc:11 mslabel:
a=ssrc:11 label:track-id-2
```

https://juejin.im/post/6844903792001974280