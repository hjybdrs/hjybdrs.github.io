---
title: 视频jitterbuffer
date: 2020-03-22 11:40:40
category: 
    - webrtc
tags:
    - webrtc
    - jitterbuffer
    - pakcetbuffer
    - referencefinder
    - framebuffer
---

# JitterBuffer

## RtpVideoStreamReceiver



## PacketBuffer

作用： 包缓存，返回数据帧(如何定义帧？)到RtpVideoStreamReceiver

{% pullquote mindmap mindmap-md %}
- [PacketBuffer]
  - size(512-2048个pkt)
  - buffer
    - sequence_buffer 保存pkt信息
    - data_buffer 保存真实pkt
  - InsertPacket() 插入数据包
  - UpdateMissingPackets() 更新丢失pkt，用于检测P帧前的gap
  - FindFrames() 查找可用帧，并回调
  - PotentialNewFrame() 查找潜在连续帧
  - PaddingReceived() 更新padding
{% endpullquote %}

```c++
void FindFrames(uint16_t seq) {
    for(最多找一圈，并且序列号是连续) {
        if(currentPacket == frame_end) {
            while(1) {
            }
            if(is_264) {
            }
            missing.erase();
            found_frames.insert();
        }
    }
}
```
可以考虑的两个点：
* nackCount
* max(min)recvtime

## ReferenceFinder

作用： 帧排队
## FrameBuffer
