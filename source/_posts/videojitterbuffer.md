---
title: videojitterbuffer
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

## ReferenceFinder

作用： 帧排队
## FrameBuffer
