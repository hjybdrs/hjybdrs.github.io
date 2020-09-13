---
title: WebrtcStats
date: 2020-09-10 11:08:05
tags:
    - webrtc
---

Google Webrtc 统计数据相关整理
<!-- more -->

# getStats

based on commit: d4089cae47334a4228b69d6bb23f2e49ebb7496e

## call-chain

<img src="./call_chain.jpg" style="zoom:50%;" />

## Field

* frameRate: receiveFps、decodeFps、dropFps、renderFps
* bps: rtpBps(ori + fec)、rtxBps、payLoadBps(%)
* transport: rtt、jitterBufferDelay、jitterBufferEmittedCount(from packet queue to jitterBufferDelay and Count)
* rtcp: pliCount、nackCount

base on these IDL:
* [RTCRtpStreamStats](https://www.w3.org/TR/webrtc-stats/#dom-rtcrtpstreamstats)
* [RTCReceivedRtpStreamStats](https://www.w3.org/TR/webrtc-stats/#dom-rtcreceivedrtpstreamstats)
* [RTCInboundRtpStreamStats](https://www.w3.org/TR/webrtc-stats/#dom-rtcinboundrtpstreamstats)
* [RTCRemoteInboundRtpStreamStats](https://www.w3.org/TR/webrtc-stats/#dom-rtcremoteinboundrtpstreamstats)
* [RTCTransportStats](https://www.w3.org/TR/webrtc-stats/#dom-rtcsctptransportstats)
* [RTCIceCandidatePairStats](https://www.w3.org/TR/webrtc-stats/#dom-rtcicecandidatepairstats)



https://juejin.im/post/6844903792001974280
https://blog.csdn.net/Chengzi_comm/article/details/89526935

## 二次开发

{% plantuml %}

package "webrtc"
{
    class VideoMediaChannel
    {
        void getStats()
    }
    class VoiceMediaChannel
    {
        void getStats()
    }

}

package "SDK"
{
    package "MediaStream"
    {
        class MediaStream
        {
            MediaEngineWrapper _engine
            void getVideoInfo()
            void getShareInfo()
            void getAudioInfo()
        }
        
        class MediaEngineWrapper
        {
            VideoMediaChannel   _video
            VideoMediaChannel   _share
            VoiceMediaChannel   _audio
            void getVideoInfo()
            void getShareInfo()
            void getAudioInfo()
        }

    }

    package "MediaEngine"
    {

        class VideoManager
        {
            MediaStream  _mediaStream
            void onStatistics()
        }

        class ShareManager
        {
            MediaStream  _mediaStream
            void onStatistics()
        }

        class AudioManager
        {
            MediaStream  _mediaStream
            void onStatistics()
        }

        class MaxMetrics
        {
            HTTP _http
        }
        
        package ELK <<Cloud>>
        {
            class HTTP
        }
        
        class ConfManager
        { 
            MediaStream  _mediaStream
            VidoeManager _videoM
            AudioManager _videoM
            ShareManager _videoM
            MaxMetrics _metrics
            void onTimer()
        }
    }

}



VideoMediaChannel --o MediaEngineWrapper
VoiceMediaChannel --o MediaEngineWrapper

MediaEngineWrapper --o MediaStream

VideoManager --o ConfManager
ShareManager --o ConfManager
AudioManager --o ConfManager

MediaStream --o VideoManager
MediaStream --o ShareManager
MediaStream --o AudioManager
MediaStream --o ConfManager
HTTP  --* MaxMetrics
MaxMetrics -left-* ConfManager

{% endplantuml %}