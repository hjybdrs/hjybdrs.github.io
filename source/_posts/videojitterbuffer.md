---
title: Videojitterbuffer
date: 2020-09-08 22:53:40
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

based on:m74(commit cc1b32545db7823b85f5a83a92ed5f85970492c9)

## RtpVideoStreamReceiver

{% plantuml %}

package "webrtc"
{
    abstract class DecodedImageCallback {
        void Decode(); 解码完成后的回调
    }

    abstract class VideoDecoder {
        void Decode();
        void RegisterDecodeCompleteCallback(DecodedImageCallback* cb);
    }

    class H264Decoder {
        void Create();
    }

    class H264DecoderImpl {
        void RegisterDecodeCompleteCallback(DecodedImageCallback* cb);
    }

    DecodedImageCallback --o H264DecoderImpl

    H264Decoder --|> VideoDecoder
    H264DecoderImpl --|> H264Decoder

    abstract class VideoSinkInterface<VideoFrame> {
        void OnFrame();
    }

    abstract class EncodedImageCallback {
        void OnEncodedImage();
    }

    abstract class NackSender {
        void SendNack();
    }

    abstract class KeyFrameRequestSender {
        void RequestKeyFrame();
    }

    abstract class OnCompleteFrameCallback {
        void OnCompleteFrame();
    }

    abstract class Syncable {

    }

    abstract class CallStatsObserver {
        void OnRttUpdate();
    }

    class PacketBuffer {

    }

    class FrameReferenceFinder {

    }

    class VCMTiming {

    }

    class RtpVideoStreamReceiver {
        PacketBuffer packetBuffer_;
        FrameReferenceFinder finder_;
        VCMTiming* vcmtiming_;
    }

    PacketBuffer --o RtpVideoStreamReceiver
    FrameReferenceFinder --o RtpVideoStreamReceiver 

    class FrameBuffer {
        VCMTiming* vcmtiming_;
    }

    class VCMGenericDecoder {
        void Decode();
        VideoDecoder decoder_;
    }

    VideoDecoder --o VCMGenericDecoder

    class VCMDecodedFrameCallback {
        void Decode(); h264解码器回调到此处，再到VideoStreamDecoder 执行FrameRender
    }

    VCMDecodedFrameCallback --|> DecodedImageCallback


    class VCMCodecDataBase {
       VCMGenericDecoder* GetDecoder(Frame, VCMDecodedFrameCallback* cb);
       void RegisterExternalDecoder();
       VCMGenericDecoder* CreateAndInitDecoder();创建并初始化解码器
       VCMGenericDecoder ptr_decoder_;
    }

    VCMGenericDecoder --o VCMCodecDataBase
    VCMDecodedFrameCallback --o VCMCodecDataBase
    VCMDecodedFrameCallback --o VideoReceiver


    class VideoReceiver {
        VCMTiming* vcmtiming_;
        VCMCodecDataBase codecBase;
        VCMDecodedFrameCallback _decodedFramecb; 
        void RegisterReceiveCallback(VCMReceiveCallback* cb);  cb 就是VideoStreamDecoder 调用设置 _decodedFramecb
        void Decode();
        void RegisterExternalDecoder(VideoDecoder* decoder);
    }

    VideoDecoder --o VideoReceiver
    VCMCodecDataBase --o VideoReceiver
    VCMTiming --o RtpVideoStreamReceiver
    VCMTiming --o FrameBuffer
    VCMTiming --o VideoReceiver

    abstract class VCMReceiveCallback {
        void FrameToRender();
        void ReceivedDecodedReferenceFrame();
    }

    class VideoStreamDecoder {
        VideoReceiver videoReceiver_;
        void FrameToRender();
        VideoSinkInterface* incoming_video_stream_; 实际上就是VideoReceiveStream
    }

    IncomingVideoStream --o VideoStreamDecoder

    VideoStreamDecoder --|> VCMReceiveCallback

    VideoReceiver --o VideoStreamDecoder

    class IncomingVideoStream {

    }

    IncomingVideoStream --|> VideoSinkInterface

    class VideoReceiverStream {
        VideoReceiverStream();
        Decode();
        Start();
        FrameBuffer frameBuffer_;
        RtpVideoStreamReceiver rtpVideoStreamReceiver_;
        VideoReceiver   videoReceiver_;
        VideoStreamDecoder videoStreamDecoder_;
        VCMTiming vctiming_;
        IncomingVideoStream incomingVideoStream_;
    }

    VideoReceiver --o VideoReceiverStream
    VideoStreamDecoder --o VideoReceiverStream
    VCMTiming --o VideoReceiverStream
    FrameBuffer --o VideoReceiverStream
    RtpVideoStreamReceiver --o VideoReceiverStream

    VideoReceiverStream --|> VideoSinkInterface
    VideoReceiverStream --|> EncodedImageCallback
    VideoReceiverStream --|> NackSender
    VideoReceiverStream --|> KeyFrameRequestSender
    VideoReceiverStream --|> OnCompleteFrameCallback
    VideoReceiverStream --|> Syncable
    VideoReceiverStream --|> CallStatsObserver
}

{% endplantuml %}

## VideoReceiver

## VideoStreamDecoder

## VCTiming

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
                // 如果不是关键帧，当前帧前面有丢失的数据包
                // 充值data_buffer 相关数据包的状态
                if(!iskeyframe && gap()){
                    reset_databuffer_status();
                }
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

## RtpFrameReferenceFinder

作用：找到每一个帧的参考帧，关键帧是自参考，后续的GOP内的每一帧都参考上一帧，GOP 内有序。

{% pullquote mindmap mindmap-md %}
- [RtpFrameReferenceFinder]
  - ManageFrame()
    - ManageFrameInternal()
    - ManageFramePidOrSeqNum()
  - PaddingReceived()
  - UpdateLastPictureIdWithPadding()
  - RetryStashedFrames()
  - ClearTo()
{% endpullquote %}

```c++
void ManageFramePidOrSeqNum() {
    // 1.插入关键帧 last_seq_num_gop(关键帧最后一个包序列号，作为下一帧的依赖)
    // 2.关键帧为空，直接缓存
    // 3.删除较老的关键帧信息，至少保存一个关键帧序列号
    // 4.如果是P帧的话，判断序列号是否和上一个连续帧的最后一个序列号相等，否则返回缓存
    // 5.更新关键帧序列的最后一个连续帧序列号，作为下一帧的依赖，和1 相互呼应
    // 6.更新最后一个序列号考虑padding 场景
}

void UpdateLastPictureIdWithPadding(seq) {
    // 1.如果当前序列号比关键帧序列号还老，返回
    // 2.获取当前seq 所依赖的关键帧信息
    // 3.如果有因padding 包存在可以使得序列号连续的，更新包的序列号
    // 4.极端情况下，清楚关键帧信息和状态
}

void RetryStashedFrames() {
    // 有两种情况可以尝试从stashed 中查找
    // a. 缓存帧遍历查找
    // b. 因padding 包的存在，可以使得序列号连续
    // 1. 从缓存帧处遍历，查看是否能找到连续的帧
}
```
## FrameBuffer

作用：GOP间 内有序。

{% pullquote mindmap mindmap-md %}
- [FrameBuffer]
  - ValidReferences() 判断帧之间的参考是否正确
    - 判断pic_id pic_id 就是当前帧最后一个包的seq
    - 判断pid 和依赖的序列号是否乱序
    - 判断所依赖的序列号是否有重复
    - 应该是判断时域分层内容
  - UpdateFrameInfoWithIncomingFrame() 更新frameinfo 和其他frame的info
    - 遍历当前帧的依赖帧
      - 如果上一次解码的帧比依赖帧更新，依赖帧永远不会被解码 rt=>false
      - 在frames 中查找依赖帧，并判断依赖帧是否连续
    - inter_layer_predicted only vp9 support
    - 未连续参考帧和未解码参考帧计数器
    - 判断依赖帧是否连续，并且更新依赖帧被哪些帧所依赖(反向依赖关系)
  - PropagateContinuity()
    - 通过BFS来进行连续性的传播，还有个依赖的就是上一层建立的反向依赖关系
  - Public
    - Start() 随着videoreceivestream start
    - Stop() 随着stop
    - SetProtectionMode()
    - Clear()
    - InsertFrame() 插入数据帧
        - 0.stats 回调
        - 1.最近连续pic_id(实际上就是seq)
        - 2.ValidReferences()
        - 3.缓存帧数量控制
        - 4.获取最近解码帧的pic_id 和 时间戳(rtp tmp)
        - 5.针对跳帧情况特殊处理，id_small&&time_new&&keyframe
        - 6.还是针对跳帧，插入会导致混乱情况，返回
        - 7.尝试插入，如果已经存在，返回
        - 8.UpdateFrameInfoWithIncomingFrame()
        - 9.判断当前帧是否因为重传导致延时(一定范围内？整个一帧)
        - 10.如果当前真的参考都到期了，计算准备返回可解码帧id
        - 11.call PropagateContinuity()
        - 12.最后唤醒解码线程并返回可以连续的帧pic_id
    - UpdateRtt() 根据rtt 调整jitterEsmitor 策略
    - NextFrame() 弹出数据帧
      - keyframe->200ms deltaframe->3000ms
      - 遍历缓存帧
        - 没有连续或者还有依赖的未解码帧
        - 当前帧并不为目前所需要的关键帧
        - 获取上一次解码帧的rtp tmp 比较筛选出跳帧的情况
{% endpullquote %}