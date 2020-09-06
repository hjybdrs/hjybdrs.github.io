---
title: 视频jitterbuffer
date: 2020-03-22 11:40:40
tags:
    - webrtc
---

# JitterBuffer

## RtpVideoStreamReceiver



## PacketBuffer

作用： 包缓存，返回数据帧(如何定义帧？)到RtpVideoStreamReceiver

主要流程：
1、InsertPacket()
2、UpdateMissingPackets()
3、FindFrames()
4、CallBack()

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

## ReferenceFinder

作用： 帧排队
## FrameBuffer
