---
title: webrtc的Metrics采集
date: 2020-03-15 11:13:40
tags: 
    - webrtc 
    - metrics 
    - c++
---

# 数据采集


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