---
title: rtprtcp
date: 2020-08-14 15:41:05
tags:
    - webrtc
---

# RTP/RTCP协议

分类：

RTP 关乎数据传输

RTCP 关乎数据传输控制



## RTP 协议和头部扩展

### 首部格式

RTP 分组的首部格式：

<img src=" ./RTP分组首部格式.png" style="zoom:67%;" />

### 包头信息

包头各个字段信息如下所示：

<img src="./RTP包头信息.png" style="zoom:67%;" />

* 版本号(V):2比特，标识rtp 版本

* 填充位(P):1比特，如果设置该位置的话，rtp包的尾部就包含附加的填充字节

* 扩展位(X):1比特，如果设置该位置的话，RTP 固定头部(12字节)后就跟着一个扩展头部

* CSRC计数器(CC):4 比特，在固定头部后跟随的CSRC 的数目

* 标记位(M):1比特，由具体的解释文档说明，对于H264来说标识一帧的结束

* 荷载类型(PT):7比特，标识RTP荷载的类型

* 序列号:2字节，每发送一个RTP 数据包，序列号增加1。接收端根据此检测丢包和重建包序列

* 时间戳:4字节，记录一个时间戳

* 同步源标识符(SSRC):4字节，随机算法生成

* 贡献源列表(CSRC):4字节，0~15个CSRC，具体数目由CSRC 计数器(CC)计算生成。

### 头部扩展

  头部扩展如下：

  <img src="./RTP头部扩展.png" style="zoom:67%;" />

如上所述，如果扩展位置为1，则一个长度可变的头扩展部分被加到RTP 固定头之后。头扩展包含16位比特的长度与，指示扩展项中32比特字的个数，但是不包含4字节扩展头的长度。为了实现特定的不同扩展，其中扩展项的前16位比特用以识别标识符或者参数。

**有且仅有一个头部扩展**



> https://blog.csdn.net/machh/article/details/51868569