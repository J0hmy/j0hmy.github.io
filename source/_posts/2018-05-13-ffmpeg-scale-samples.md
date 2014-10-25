---
title: ffmpeg scale samples
date: 2018-05-13 17:49:16
tags: ffmpeg
---

ffmpeg 音量增益

```c
// https://www.ffmpeg.org/doxygen/2.7/af__volume_8c_source.html

// static inline int16_t av_clip_int16(int a)
// {
//     if ((a+0x8000U) & ~0xFFFF) return (a>>31) ^ 0x7FFF;
//     else                      return a;
// }

static inline void scaleSamplesS16Volume(uint8_t *dst, int nb_samples, double adjustVolumeValue) {
    int16_t *smp_dst       = (int16_t *)dst;

    int volume = (int)(adjustVolumeValue * 256 + 0.5);
    // volume < if (vol->volume_i < 0x1000000)
    printf("\r====%d %lf   ", volume, adjustVolumeValue);
    if (volume < 0x1000000) {
        // vol->scale_samples = scale_samples_u8_small;
        for (int i = 0; i < nb_samples; ++i) {
            smp_dst[i] = av_clip_int16((smp_dst[i] * volume + 128) >> 8);
        }
    } else {
        // vol->scale_samples = scale_samples_u8;
        for (int i = 0; i < nb_samples; ++i) {
            smp_dst[i] = av_clip_int16(((int64_t)smp_dst[i] * volume + 128) >> 8);
        }
    }
}

```