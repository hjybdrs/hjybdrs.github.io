---
title: getStats
date: 2020-08-02 11:08:05
tags:
---

# getStats()

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