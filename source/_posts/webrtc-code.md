---
title: webrtc-code
date: 2020-09-01 09:47:05
tags:
    - webrtc
---

# webrtc-code

## 序列号判断

```c++
// 如果seq2 比seq1 要老的话，返回真
// 1 2 3 4 5 6 7 8 
// seq1 = 6 seq2 = 4 返回真
bool AheadOf(uint16_t seq1, uint16_t seq2);
```
