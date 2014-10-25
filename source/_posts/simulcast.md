---
title: Simulcast
date: 2021-06-16 17:26:05
tags: webrtc
---

Simulcast, 效果演示可以参考这个项目 [simulcast-playground][simulcast-playground]

### H264EncoderImpl 实现

就 H264 而言, H264EncoderImpl 实现了Simulcast 功能, 为了使用 H264EncoderImpl, 需要webrtc编译ffmpeg和openh264, 代码可以参考 https://github.com/johzzy/webrtc-mirror/tree/feature/h264

```
// modules/video_coding/codecs/h264/h264_encoder_impl.cc
int32_t H264EncoderImpl::InitEncode(const VideoCodec* inst,
                                    const VideoEncoder::Settings& settings) {
    ...
    for (int i = 0, idx = number_of_streams - 1; i < number_of_streams;
       ++i, --idx) {
    ISVCEncoder* openh264_encoder;
    // Create encoder.
    if (WelsCreateSVCEncoder(&openh264_encoder) != 0) {
      // Failed to create encoder.
      RTC_LOG(LS_ERROR) << "Failed to create OpenH264 encoder";
      RTC_DCHECK(!openh264_encoder);
      Release();
      ReportError();
      return WEBRTC_VIDEO_CODEC_ERROR;
    }
    RTC_DCHECK(openh264_encoder);
    if (kOpenH264EncoderDetailedLogging) {
      int trace_level = WELS_LOG_DETAIL;
      openh264_encoder->SetOption(ENCODER_OPTION_TRACE_LEVEL, &trace_level);
    }
    // else WELS_LOG_DEFAULT is used by default.

    // Store h264 encoder.
    encoders_[i] = openh264_encoder;
        ...
```

从 `H264EncoderImpl::InitEncode` 函数可以看到, 真正负责编码工作的对象是 `ISVCEncoder* openh264_encoder;`, `Simulcast` 和 `SVC` 都可以由它完成.

在 `Android/iOS` 等移动端, `H264EncoderImpl` 实现的 `Simulcast` 因 `OpenH264` 软件编码的缘故, 往往有CPU算力和电池续航方面的不足. 

另外, `WebRTC iOS`编译 `FFmpeg` 和 `OpenH264` 依然是一个巨坑. 那么有没有利用硬件编码实现的Simulcast呢.

### SimulcastEncoderAdapter 实现

`SimulcastEncoderAdapter` 代码详见 `media/engine/simulcast_encoder_adapter.cc`, 它本身是一个通用的适配器, 给SimulcastEncoderAdapter 传入相应的编码器工厂，就可以组成一个simulcast的编码器, 

```
SimulcastEncoderAdapter::SimulcastEncoderAdapter(VideoEncoderFactory* factory,
                                                 const SdpVideoFormat& format)
    : SimulcastEncoderAdapter(factory, nullptr, format) {}
```


具体来说, 在 `Android` 平台,可以给 `SimulcastEncoderAdapter` 传入 `mediacodec` 的 `h264` 编码器开启硬编,在 `iOS` 平台则可以传入 `videotoolbox`. 当然,你可以 SimulcastEncoderAdapter 做到不同层级的视频使用不同的编码器, 混合使用.

在 chrome 中 h264 的 simulcast 就是这样实现的，在 `chrome://webrtc-internals` 如果看到这样的字符串,就是 `SimulcastEncoderAdapter` 实现的

```
encoderImplementation SimulcastEncoderAdapter (OpenH264, ExternalEncoder, ExternalEncoder)
```

android 平台利用 SimulcastEncoderAdapter 开启 simulcast 的一种实例

```
// SimulcastEncoderAdapter::GetEncoderInfo().implementation_name 输出
// 其中 OpenH264 编码小流, HWEncoder 编码大流
SimulcastEncoderAdapter (OpenH264, HWEncoder)
```



### 如何使用 SimulcastEncoderAdapter

可以参考 chromium blink 的使用方式,具体代码见 `third_party/blink/renderer/platform/peerconnection/video_codec_factory.cc`

#### 创建相应工厂
仿照 blink 创建 EncoderAdapter 和 CreateVideoEncoderAdapterFactory
```

// @see 
template <typename Factory>
bool IsFormatSupported(const Factory* factory,
                       const webrtc::SdpVideoFormat& format) {
  return factory && IsFormatSupported(factory->GetSupportedFormats(), format);
}

// Merge |formats1| and |formats2|, but avoid adding duplicate formats.
std::vector<webrtc::SdpVideoFormat> MergeFormats(
    std::vector<webrtc::SdpVideoFormat> formats1,
    const std::vector<webrtc::SdpVideoFormat>& formats2) {
  for (const webrtc::SdpVideoFormat& format : formats2) {
    // Don't add same format twice.
    if (!IsFormatSupported(formats1, format))
      formats1.push_back(format);
  }
  return formats1;
}

// This class combines a hardware factory with the internal factory and adds
// internal SW codecs, simulcast, and SW fallback wrappers.
class EncoderAdapter : public webrtc::VideoEncoderFactory {
 public:
  explicit EncoderAdapter(
      std::unique_ptr<webrtc::VideoEncoderFactory> hardware_encoder_factory)
      : hardware_encoder_factory_(std::move(hardware_encoder_factory)) {}

  webrtc::VideoEncoderFactory::CodecInfo QueryVideoEncoder(
      const webrtc::SdpVideoFormat& format) const override {
    const webrtc::VideoEncoderFactory* factory =
        IsFormatSupported(hardware_encoder_factory_.get(), format)
            ? hardware_encoder_factory_.get()
            : &software_encoder_factory_;
    return factory->QueryVideoEncoder(format);
  }

  std::unique_ptr<webrtc::VideoEncoder> CreateVideoEncoder(
      const webrtc::SdpVideoFormat& format) override {
    const bool supported_in_software =
        IsFormatSupported(&software_encoder_factory_, format);
    const bool supported_in_hardware =
        IsFormatSupported(hardware_encoder_factory_.get(), format);

    if (!supported_in_software && !supported_in_hardware)
      return nullptr;

    if (absl::EqualsIgnoreCase(format.name.c_str(),
                                         cricket::kVp9CodecName) ||
        absl::EqualsIgnoreCase(format.name.c_str(),
                                         cricket::kAv1CodecName)) {
      // For VP9 and AV1 we don't use simulcast.
      // return software_encoder_factory_.CreateVideoEncoder(format);
      return hardware_encoder_factory_->CreateVideoEncoder(format);
    }

    if (!supported_in_hardware || !hardware_encoder_factory_.get()) {
      return std::make_unique<webrtc::SimulcastEncoderAdapter>(
          &software_encoder_factory_, nullptr, format);
    } else if (!supported_in_software) {
      return std::make_unique<webrtc::SimulcastEncoderAdapter>(
          hardware_encoder_factory_.get(), nullptr, format);
    }

    return std::make_unique<webrtc::SimulcastEncoderAdapter>(
        hardware_encoder_factory_.get(), &software_encoder_factory_, format);
  }

  std::vector<webrtc::SdpVideoFormat> GetSupportedFormats() const override {
    std::vector<webrtc::SdpVideoFormat> software_formats =
        software_encoder_factory_.GetSupportedFormats();
    return hardware_encoder_factory_
               ? MergeFormats(software_formats,
                              hardware_encoder_factory_->GetSupportedFormats())
               : software_formats;
  }

 private:
  webrtc::InternalEncoderFactory software_encoder_factory_;
  const std::unique_ptr<webrtc::VideoEncoderFactory> hardware_encoder_factory_;
};

std::unique_ptr<VideoEncoderFactory> CreateVideoEncoderAdapterFactory(
  std::unique_ptr<webrtc::VideoEncoderFactory> hardware_encoder_factory) {
  return std::make_unique<EncoderAdapter>(std::move(hardware_encoder_factory));
}
```

#### 使用工厂

```
//  ios 使用
        auto video_encoder_factory = std::move(native_encoder_factory);
        return [self initWithNativeAudioEncoderFactory:webrtc::CreateBuiltinAudioEncoderFactory()
                             nativeAudioDecoderFactory:webrtc::CreateBuiltinAudioDecoderFactory()
                             nativeVideoEncoderFactory:webrtc::CreateVideoEncoderAdapterFactory(std::move(video_encoder_factory))
                             nativeVideoDecoderFactory:webrtc::CreateBuiltinVideoDecoderFactory()
                                     audioDeviceModule:[self audioDeviceModule]
                                 audioProcessingModule:nullptr];
// android 使用
  cricket::MediaEngineDependencies media_dependencies;
  media_dependencies.task_queue_factory = dependencies.task_queue_factory.get();
  media_dependencies.adm = std::move(audio_device_module);
  media_dependencies.audio_encoder_factory = std::move(audio_encoder_factory);
  media_dependencies.audio_decoder_factory = std::move(audio_decoder_factory);
  media_dependencies.audio_processing = std::move(audio_processor);
  auto video_encoder_factory =
      absl::WrapUnique(CreateVideoEncoderFactory(jni, jencoder_factory));
  media_dependencies.video_encoder_factory = 
        CreateVideoEncoderAdapterFactory(std::move(video_encoder_factory));
  media_dependencies.video_decoder_factory =
      absl::WrapUnique(CreateVideoDecoderFactory(jni, jdecoder_factory));
  dependencies.media_engine =
      cricket::CreateMediaEngine(std::move(media_dependencies));

  rtc::scoped_refptr<PeerConnectionFactoryInterface> factory =
      CreateModularPeerConnectionFactory(std::move(dependencies));
```





参考: https://blog.jianchihu.net/webrtc-research-simulcast-layer-change.html


[simulcast-playground]: https://fippo.github.io/simulcast-playground/
