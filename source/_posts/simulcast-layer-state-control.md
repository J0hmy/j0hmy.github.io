---
title: simulcast layer state control
date: 2021-06-23 17:08:04
tags:
---

simulcast 层级状态控制

webrtc通过 StreamContext 表示simulcast的层级概念

如果需要暂停simulcast某层的工作,设置 layer_context.set_is_paused(true)即可,webrtc代码引用如下
```
  for (StreamContext& layer_context : stream_contexts_) {
    int stream_idx = layer_context.stream_idx();
    uint32_t stream_bitrate_kbps =
        parameters.bitrate.GetSpatialLayerSum(stream_idx) / 1000;

    // Need a key frame if we have not sent this stream before.
    if (stream_bitrate_kbps > 0 && layer_context.is_paused()) {
      layer_context.set_is_keyframe_needed();
    }
    layer_context.set_is_paused(stream_bitrate_kbps == 0);
    ...
  }
```

设置后,webrtc会在以下逻辑中使用它
```
  for (auto& layer : stream_contexts_) {
    // Don't encode frames in resolutions that we don't intend to send.
    if (layer.is_paused()) {
      continue;
    }
    ...
  }
```

有上述代码可知, parameters.bitrate.GetSpatialLayerSum(stream_idx) 控制着 layer_context 的状态 state(actived/paused), 进而通过控制 layer_context 控制编码器工作 

那么从哪里修改 parameters.bitrate 呢? 外部接口如何操作它呢?

(待续未完 ...)


MediaStreamTrack.SetEnabled
JNI_MediaStreamTrack_SetEnabled
MediaStreamTrackInterface
VideoTrackInterface
VideoTrack
VideoTrackSource::AddOrUpdateSink(video_source_->AddOrUpdateSink)
AdaptedVideoTrackSource::AddOrUpdateSink
VideoBroadcaster::AddOrUpdateSink
VideoSourceBase::AddOrUpdateSink, UpdateWants