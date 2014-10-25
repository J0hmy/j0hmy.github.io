---
title: Field Trails
date: 2021-06-01 19:34:12
tags: webrtc
---


## Field Trials 介绍
field trials （试用特性）是Chrome 的一种特性开关机制，WebRTC沿用了它。其本质是一种特定格式的字符串，格式如下：

    "FieldTrialName1/Enabled/FieldTrialName2/Disabled/FieldTrialName3/Disabled/"

### 最简格式为空字符串

    ""

### 基本单元
    
    "FieldTrialName/FieldTrialValue/"

### FieldTrialValue的有效值
常见的FieldTrialValue取值有 Enabled 和 Disabled，一些例外的还有形如 Default，EnabledByFlag_2SL3TL，比如

    WebRTC-H264Simulcast/Enabled/
    WebRTC-LegacySimulcastLayerLimit/Disabled/
    WebRTC-SupportVP9SVC/Default/
    WebRTC-SupportVP9SVC/EnabledByFlag_2SL3TL/
    WebRTC-Vp9InterLayerPred/Enabled,inter_layer_pred_mode:off/
    WebRTC-Vp9InterLayerPred/Enabled,inter_layer_pred_mode:on/
    WebRTC-Vp9InterLayerPred/Enabled,inter_layer_pred_mode:onkeypic/
    WebRTC-VP8-Forced-Fallback-Encoder-v2/Enabled-1,2,34567/
    // 开启 Audio-NetEqExtraDelay 值为 120ms
    WebRTC-Audio-NetEqExtraDelay/Enabled-120/

特殊例子中FieldTrialValue的具体含义取决于相应代码的解释，一般以Enabled开头


在chrome浏览器中，可以打开 chrome://flags 查看 FieldTrials


## WebRTC利用它完成了什么？

    constexpr char kUseLegacySimulcastLayerLimitFieldTrial[] =
        "WebRTC-LegacySimulcastLayerLimit";
    
    
    if (!absl::StartsWith(trials.Lookup(kUseLegacySimulcastLayerLimitFieldTrial), "Disabled")) {
    // Case A: kUseLegacySimulcastLayerLimitFieldTrial is Enabled
    } else {
    // Case B: kUseLegacySimulcastLayerLimitFieldTrial is Disabled
    }

当我们设置的 fieldtrials 中包含 "WebRTC-LegacySimulcastLayerLimit/Disabled" 则走到 Case B，包含 "WebRTC-LegacySimulcastLayerLimit/Enabled" 则走到 Case A。

思考：都不包含会走A还是B？



## FieldTrials 如何使用？

    // c++ @see https://github.com/johzzy/webrtc-mirror/blob/master/system_wrappers/include/field_trial.h#L83
    webrtc::field_trial::InitFieldTrialsFromString(const char* trials_string);
    // 举例
    webrtc::field_trial::InitFieldTrialsFromString("Audio/Enabled/Video/Disabled/");
 
 
 
    // java @see https://github.com/johzzy/webrtc-mirror/blob/master/sdk/android/api/org/webrtc/PeerConnectionFactory.java#L99
    public PeerConnectionFactory.InitializationOptions.Builder setFieldTrials(String fieldTrials)
    // 举例
    String fieldTrials = "WebRTC-LegacySimulcastLayerLimit/Disabled/";
    PeerConnectionFactory.initialize(
            PeerConnectionFactory.InitializationOptions.builder(context)
                    .setFieldTrials(fieldTrials)
                    .setEnableInternalTracer(true)
                    .createInitializationOptions());
    // oc @see https://github.com/johzzy/webrtc-mirror/blob/master/sdk/objc/api/peerconnection/RTCFieldTrials.h#L33
    void RTCInitFieldTrialDictionary(NSDictionary<NSString *, NSString *> *fieldTrials) {
    if (!fieldTrials) {
        RTCLogWarning(@"No fieldTrials provided.");
        return;
    }
    // Assemble the keys and values into the field trial string.
    // We don't perform any extra format checking. That should be done by the underlying WebRTC calls.
    NSMutableString *fieldTrialInitString = [NSMutableString string];
    for (NSString *key in fieldTrials) {
        NSString *fieldTrialEntry = [NSString stringWithFormat:@"%@/%@/", key, fieldTrials[key]];
        [fieldTrialInitString appendString:fieldTrialEntry];
    }
    size_t len = fieldTrialInitString.length + 1;
    gFieldTrialInitString.reset(new char[len]);
    if (![fieldTrialInitString getCString:gFieldTrialInitString.get()
                                maxLength:len
                                encoding:NSUTF8StringEncoding]) {
        RTCLogError(@"Failed to convert field trial string.");
        return;
    }
    webrtc::field_trial::InitFieldTrialsFromString(gFieldTrialInitString.get());
    }
    
 
    举例
    NSDictionary *fieldTrials = @{};
    RTCInitFieldTrialDictionary(fieldTrials);
    
    
    // 命令行
    --force-fieldtrials=WebRTC-H264Simulcast/Enabled/


## FieldTrials 实现

    https://github.com/johzzy/webrtc-mirror/blob/master/system_wrappers/include/field_trial.h
    https://github.com/johzzy/webrtc-mirror/blob/master/system_wrappers/source/field_trial_unittest.cc


## FieldTrials 应用示例

使用 WebRTC-PcFactoryDefaultBitrates 控制码率

    WebRTC-PcFactoryDefaultBitrates/Enabled,min:60,max:4000,start:600/


码率控制也可以通过sdp进行（相比FieldTrials，sdp设置的优先级更高）

    a=fmtp:102 level-asymmetry-allowed=1;packetization-mode=1;x-google-min-bitrate=400;x-google-max-bitrate=600;profile-level-id=42e01f




