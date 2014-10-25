---
title: simulcast support in srs
date: 2021-06-18 18:50:41
tags: webrtc
---

最近弄明白了 Simulcast, 就给 SRS 加了 Simulcast 功能支持, 不过是 Chrome 独有的 SDP munging 风格的 Simulcast, 以后了解清楚了标准的 Simulcast 格式再完善一下.

PR 详见 https://github.com/ossrs/srs/pull/2420


## 发布端

使用  `numberOfSimulcastLayers` 标示 Simulcast 层级数, 可选的有效值为 1,2,3; 其中1为默认值, 此外的其他值为无效值, 若误用会重置为默认1.

`numberOfSimulcastLayers = 1` 相当于 `Simulcast 流` 退化为 `单流`

    webrtc://127.0.0.1/live/livestream?numberOfSimulcastLayers=1  (相当于单流)
    webrtc://127.0.0.1/live/livestream?numberOfSimulcastLayers=2  (2 层 Simulcast 流)
    webrtc://127.0.0.1/live/livestream?numberOfSimulcastLayers=3  (3 层 Simulcast 流)

## 播放端

播放端使用 `layer` 选择哪一层视频流,可选的有效值为 0,1,2,3,...
如果 `layer` 对应的视频流无法播放,则退化为一般场景的播放端.

    low: webrtc://127.0.0.1/live/livestream?layer=0
    mid: webrtc://127.0.0.1/live/livestream?layer=1
    high: webrtc://127.0.0.1/live/livestream?layer=2
    other1: webrtc://127.0.0.1/live/livestream?layer=1,2
    other2: webrtc://127.0.0.1/live/livestream?layer=1,2&foo=bar