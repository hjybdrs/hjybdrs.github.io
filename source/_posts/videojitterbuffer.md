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

    abstract class OnAssembledFrameCallback {
        void OnAssembledFrame();
    }

    package "jitterbuffer" {
        class VCMTiming {

        }

        class FrameBuffer {
            VCMTiming* vcmtiming_;
        }

        class PacketBuffer {
            OnAssembledFrameCallback* cb; actually RtpVideoStreamReceiver
            void InsertPacket();
            void FindFrames();
        }

        OnAssembledFrameCallback --o PacketBuffer

        class FrameReferenceFinder {

        }

        class RtpVideoStreamReceiver {
            PacketBuffer packetBuffer_;
            FrameReferenceFinder finder_;
            VCMTiming* vcmtiming_;
            void OnRtpPacket(); 从demux 接收数据
            void OnReceivedPayloadData(); 从rtp包层面解开数据包
        }
    }

    abstract class RtpPacketSinkInterface {
        void OnRtpPacket();
    }

    class RtxReceiveStream {
        RtpPacketSinkInterface* media_sink; 实际上就是RtpVideoStreamReceiver
        void OnRtpPacket(); 从demux 接收数据
    }

    RtpVideoStreamReceiver --|> OnAssembledFrameCallback
    RtxReceiveStream --|> RtpPacketSinkInterface
    RtpVideoStreamReceiver --|> RtpPacketSinkInterface

    PacketBuffer --o RtpVideoStreamReceiver
    FrameReferenceFinder --o RtpVideoStreamReceiver 

    

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
和VCMtiming 关系： 没有直接关系

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
                // 重置data_buffer 相关数据包的状态
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
和VCMtiming 关系： 没有直接关系

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
// 在调用这个前插入了一个picture_id 针对h264 是否存在？
void ManageFramePidOrSeqNum() {
    // map<last_seq, pair<seq, seq>> last_seq_num_gop_
    // 1. 如果是关键帧，则插入关键帧 到last_seq_num_gop_
    // 2. 如果last_seq_num_gop_ 为空，则缓存这个帧，表明当前并没有关键帧存在
    // 3. 删除较老的关键帧信息(当前seq-100)，至少保存一个关键帧序列号
    // 4. 如果当前帧所依赖的关键帧已经不存在了，直接丢弃。并找到当前帧所依赖的关键帧的序列号
    // 5. 如果是P帧的话，判断帧第一个包序列号是否和对应关键帧的最后一个序列号(会随着帧连续不断修改)相等，否则返回缓存。不相等说明中间有gap
    // 6. 更新当前插入帧的VideoFrameLayer中的pic_id 为最后一个包seq，更新是否需要依赖，并且依赖的帧的信息
    // 7. 更新当前帧所依赖关键帧序列号，作为下一帧的依赖，和1 相互呼应(在此已经保证是连续的了，5已说明)
    // 8. UpdateLastPictureIdWithPadding() last_picture_id_ 在264 中没有使用
    // 9. 更新最后一个序列号考虑padding 场景
}

void UpdateLastPictureIdWithPadding(seq) {
    // 1.如果当前序列号比last_seq_num_gop_ 中第一个关键帧序列号还老，返回。否则返回当前帧所依赖的关键帧
    // 2.如果有因padding 包存在可以使得序列号连续的，更新包的序列号
    // 3.极端情况下，清除关键帧信息和状态
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
  - UpdateFrameInfoWithIncomingFrame() 更新frameinfo 和其他frame的info
  - PropagateContinuity()
    - 通过BFS来进行连续性的传播，当前帧连续可能会引发依赖当前帧的其他帧连续，一网打尽
  - PropagateDecodability()
    - 当前帧可以解码，遍历所有依赖当前帧的数据，减少decodeable 次数，会为后续的是否可解码做判断
  - Public
    - Start() 随着videoreceivestream start
    - Stop() 随着stop
    - SetProtectionMode()
    - Clear()
    - InsertFrame() 插入数据帧
    - UpdateRtt() 根据rtt 调整jitterEsmitor 策略
    - NextFrame() 弹出数据帧
{% endpullquote %}

```c++

void NextFrame() {
    // 1. do-while()
      // a. 遍历insertFrame 插入的帧数据
        // 一. 如果当前帧非连续或者依赖的解码数大于0 ，跳过
        // 二. 如果需要关键帧，但是当前帧却不是关键帧 ，跳过
        // 三. 如果已经有解码帧，但是当前帧的时间比最近解码帧的时间还老 ，跳过 
        // 四. vp9 support 暂不考虑
        // 五. 264只有一个完整帧，only one superframe and last_layer_complete true
        // 六. 设置有效的渲染时间，并判断是否解码耗时过长，否则返回有效可解码帧数据
        // 七. 设置frame_to_decode 帧信息
    // 2. 遍历frame_to_decode 帧信息，并调用propagateDecodeablity() 然后设置frame_out 信息，根据当前帧是否被nack 过进行相关设置
    // 3. 返回帧数据
}

bool ValidReferences() {
    // 1. 判断pic_id，pic_id 就是当前帧最后一个包的seq
    // 2. 判断pic_id 和依赖的seq是否符合规定
      // a. 如果依赖的seq 比当前帧的seq 还大，不符合
      // b. 如果连续依赖的seq 之间相等，不符合
    // 3. 应该是判断时域分层内容
}

int InsertFrame() {
    // 1. 统计回调
    // 2. 最近连续帧pic_id(当前帧的最后一个seq)
    // 3. ValidReferences()帧有效性判断
    // 4. 如果当前帧数超过最大容量(800 帧)
       // a. 如果当前关键帧，清空队列，继续
       // b. 非关键帧，直接返回最近连续帧seq 
           // 1. 直接返回是否有丢弃?
           // 2. 直接丢弃会不会有问题？
    // 5. 获取最近解码帧seq 和解码时间戳(调用NextFrame时候更新),如果当前帧seq 比最近解码帧seq 小
       // a. 如果是关键帧，并且当前帧的时间更新，清空队列，继续
       // b. 其他情况，直接返回最近连续帧seq 直接丢弃会不会有问题？
    // 6. 出现跳帧情况，比第一个小，比最后一个大的情况出现，清空，继续
    // 7. 尝试插入，如果之前存在，直接返回最近解码帧的seq
    // 8. UpdateFrameInfoWithIncomingFrame()
    // 9. 判断当前帧是否因为重传导致了延时，nack_count > 0 更新timing(TimeStamp(rtpTimestamp), ReceivedTime(本机收到数据包时间)) 时间都是在PacketBuffer 返回RtpFrameObject 中生成
    // 10.如果当前帧的参考帧都到齐了
       // a. PropagateContinuity()
       // b. 计算最大限度可解码帧seq 
    // 11. 唤醒解码线程，并返回最大限度可解码帧seq

}

bool UpdateFrameInfoWithIncomingFrame() {
    // 1. 获取最近解码帧的信息，并判断当前帧的pic_id 要大于最近解码帧pic_id
    // 2. 遍历当前帧所依赖的帧数据
       // a. 如果最近已经有解码帧，并且最近解码帧大于依赖帧序列号(表明所依赖的帧已经被解码)
          // 1. 如果依赖的帧并不在解码帧历史中存在，表明当前帧永远不可能被解码了，返回false
       // b. 否则(要么从没解码过，或者所依赖的帧比已经解码的帧还要新) 查找依赖帧是否存在已经是否连续，并插入到依赖的队列中
    // 3. inter_layer_predicted only support vp9
    // 4. 统计当前帧所依赖帧的总数，以及依赖帧解码的总数
    // 5. 遍历查找依赖帧的队列，并根据依赖帧的连续性更新当前帧的信息，已经依赖帧信息(主要是被哪些帧所依赖)
    // 6. return true
}
```