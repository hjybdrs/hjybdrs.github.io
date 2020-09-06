---
title: PeerConnection
date: 2020-08-02 17:42:04
tags:
    - PeerConnection
    - webrtc
---


# PeerConnection

based on commit: d4089cae47334a4228b69d6bb23f2e49ebb7496e
useful file: 
* conductor.cc(examples/peerconnection/client)
* simple_peer_connection.cc(examples/unityplugin)

## 拉取webrtc 代码

拉取分支
```shell
git clone https://chromium.googlesource.com/external/webrtc

#设置git config
#在 .git/config 文件中的  `[remote "origin"]` 节中添加以下内容：
# fetch = +refs/branch-heads/*:refs/remotes/origin/*

git pull

# 拉取远端tags

git fetch --tags
```



{% plantuml %}

package "cricket"
{
    abstract class MediaChannel {
        bool AddSendStream(const StreamParams& sp);
        bool RemoveSendStream(uint32_t ssrc);
        bool AddRecvStream(const StreamParams& sp);
        bool RemoveRecvStream(uint32_t ssrc);
    }

    abstract class VideoMediaChannel {

    }

    VideoMediaChannel --|> MediaChannel

    abstract class VideoReceiveStream {

    }

    class WebRtcVideoReceiveStream {
        VideoReceiveStream* stream_;
    }

    VideoReceiveStream --o WebRtcVideoReceiveStream

    class VideoReceiveStream2 {
        some values; 真实的videoreceivestream
    }

    VideoReceiveStream2 --|> VideoReceiveStream

    class WebRtcVideoSendStream {

    }

    class WebRtcVideoChannel {
        bool AddRecvStream(); // 真实的创建
        bool AddSendStream(); // 真实的创建
        std::map<uint32_t, WebRtcVideoReceiveStream*> receive_streams_
        std::map<uint32_t, WebRtcVideoSendStream*> send_streams_
        
    }

    WebRtcVideoReceiveStream --o WebRtcVideoChannel
    WebRtcVideoSendStream --o WebRtcVideoChannel

    WebRtcVideoChannel --|> VideoMediaChannel

    abstract class VideoEngineInterface {
        VideoMediaChannel* CreateMediaChannel() = 0;
    }

    class WebRtcVideoEngine {
        VideoMediaChannel* CreateMediaChannel();
    }

    VideoMediaChannel --o VideoEngineInterface
    VideoMediaChannel --o WebRtcVideoEngine

    WebRtcVideoEngine --|> VideoEngineInterface

    abstract class MediaEngineInterface {
        VoiceEngineInterface& voice() = 0;
        VideoEngineInterface& video() = 0;
    }

    class CompositeMediaEngine {
        VoiceEngineInterface& voice() override;
        VideoEngineInterface& video() override;   
    }

    VideoEngineInterface --o CompositeMediaEngine
    CompositeMediaEngine --|> MediaEngineInterface

    abstract class BaseChannel {
        MediaChannel* media_channel();
        std::unique_ptr<MediaChannel> media_channel_;
        bool AddRecvStream_w(const StreamParams& sp);调用media_channel 的AddRecvStream()
        bool SetRemoteContent_w(); 
        bool SetRemoteContent();使用std::bind(SetRemoteContent_w,this) 实际上this 是videoChannel或者voiceChannel
    }

    MediaChannel --* BaseChannel

    class VideoChannel {
        VideoMediaChannel* media_channel();
        bool SetRemoteContent_w(); 设置远端的sdp 信息，获取ssrc 等内容来触发创建videoRecvStream
        bool UpdateRemoteStreams_w();
        bool AddRecvStream_w(); 真实创建
    }

    VideoChannel --|> BaseChannel
}

package "webrtc"
{

    class ChannelManager {
        MediaEngineInterface* media_engine();
        MediaEngineInterface* media_engine_;
        VideoChannel* CreateVIdeoChannel(); media_engine->video 会先创建一个VideoMediaChannel(实际上是WebRtcVideoChannel), 该对象作为参数创建VideoChannel
    }

    VideoChannel --o ChannelManager

    MediaEngineInterface --o ChannelManager

    class PeerConnectionInterface {
        
    }

    class PeerConnectionFactory {
        ChannelManager*         channel_manager_;
        MediaEngineInterface    media_engine_; 最终move 到ChannelManager
        PeerConnectionInterface* CreatePeerConnection();
    }

    PeerConnectionInterface --o PeerConnectionFactory

    ChannelManager --o PeerConnectionFactory

    class PeerConnectionInterval {

    }

    PeerConnectionInterval --|> PeerConnectionInterface

    class PeerConnection
    {
        void DoSetRemoteDescription()
        void ApplyLocalDescription(); 会调用CreateChannels() 方法
        void ApplyRemoteDescription();
        void CreateChannels(SessionDescription);根据sdp 的内容来创建音视频、data channel
        void UpdateSessionState(); 
        VideoChannel* CreateVideoChannel(const std::string& mid);
        void PushdownMediaDescription();// 调用channel的SetRemoteContent() SetLocalContent() 方法
        void getStats();   
        Call                    call_;
        PeerConnectionFactory   factory_;
    }

    PeerConnection --|> PeerConnectionInterval
    ChannelManager --o PeerConnection

}

{% endplantuml %}