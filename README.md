# Android FFmpeg Prebuilt

[![GitHub stars](https://img.shields.io/github/stars/hzw1199/Android-FFmpeg-Prebuilt?style=social)](https://github.com/hzw1199/Android-FFmpeg-Prebuilt)
[![FFmpeg](https://img.shields.io/badge/FFmpeg-8.0.1-green)](https://ffmpeg.org/)
[![Platform](https://img.shields.io/badge/Platform-Android-brightgreen)](https://developer.android.com/)
[![ABI](https://img.shields.io/badge/ABI-arm64--v8a-blue)]()
[![License](https://img.shields.io/badge/License-LGPL--2.1-orange)](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html)

**[中文文档](README_CN.md)**

Prebuilt FFmpeg binaries for Android — ready to use, no compilation needed.

Provides `ffmpeg` / `ffprobe` executables and a single `libffmpeg.so` shared library with all core modules merged in.

---

## Features

- **FFmpeg 8.0.1** — latest major release
- **Android arm64-v8a (aarch64)** — targeting modern 64-bit devices, minSdkVersion 28
- **Single shared library** — `libffmpeg.so` merges libavcodec, libavformat, libavfilter, libavutil, libavdevice, libswresample, libswscale, all 7 core libraries into one `.so`
- **Standalone executables** — `ffmpeg` and `ffprobe` can be pushed to device and run directly via shell
- **C/C++ headers included** — integrate into your Android NDK project
- **Pure LGPL build** — `--disable-gpl --disable-nonfree`, safe for commercial use
- **MediaCodec hardware acceleration** — H.264 / HEVC / MPEG-4 hardware decoding via Android MediaCodec
- **16 KB page size compatible** — meets [Google Play's 16 KB page size requirement](https://developer.android.com/guide/practices/page-sizes) for apps targeting Android 15+
- **Size-optimized** — built with `--enable-small -Os`

## File Structure

```
ffmpeg-8.0.1/
├── bin/
│   ├── ffmpeg           # CLI executable (15 MB)
│   └── ffprobe          # CLI executable (14 MB)
├── libffmpeg.so         # Merged shared library — all 7 modules in one .so (74 MB)
├── include/
│   ├── libavcodec/
│   ├── libavdevice/
│   ├── libavfilter/
│   ├── libavformat/
│   ├── libavutil/
│   ├── libswresample/
│   └── libswscale/
└── examples/            # Official FFmpeg example source code
```

## Build Configuration

### Encoders & Decoders

All LGPL-compatible encoders and decoders enabled, including MediaCodec hardware decoders:

| MediaCodec Decoder | Description |
|---|---|
| h264_mediacodec | H.264 hardware decoding via Android MediaCodec |
| hevc_mediacodec | HEVC hardware decoding via Android MediaCodec |
| mpeg4_mediacodec | MPEG-4 hardware decoding via Android MediaCodec |

### Other Capabilities

| Category | Status |
|---|---|
| Muxers / Demuxers | All enabled |
| Protocols | All enabled (network included) |
| Filters | All enabled |
| Parsers / BSFs | All enabled |
| Hardware Acceleration | MediaCodec + JNI enabled |
| iconv | Enabled |

### libffmpeg.so Included Libraries

`libffmpeg.so` is linked from all 7 static libraries:

| Library | Description |
|---|---|
| libavdevice | Device capture & playback |
| libavfilter | Audio/video filter framework |
| libavcodec | Audio/video codec encoding & decoding |
| libavformat | Muxing & demuxing (container formats) |
| libswresample | Audio resampling & format conversion |
| libswscale | Video scaling & pixel format conversion |
| libavutil | Common utility functions |

## Quick Start

### Use as CLI tool on device

```bash
adb push ffmpeg-8.0.1/bin/ffmpeg /data/local/tmp/
adb shell chmod +x /data/local/tmp/ffmpeg
adb shell /data/local/tmp/ffmpeg -version
```

### Use libffmpeg.so in Android NDK project

**CMakeLists.txt**

```cmake
add_library(ffmpeg SHARED IMPORTED)
set_target_properties(ffmpeg PROPERTIES
    IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/libs/${ANDROID_ABI}/libffmpeg.so
)
target_include_directories(your_target PRIVATE ${CMAKE_SOURCE_DIR}/ffmpeg-8.0.1/include)
target_link_libraries(your_target ffmpeg)
```

## Build Information

| Item | Value |
|---|---|
| FFmpeg Version | 8.0.1 |
| Target Platform | Android |
| Target ABI | arm64-v8a (aarch64) |
| minSdkVersion | 28 (Android 9.0) |
| NDK Version | r28 |
| Toolchain | LLVM / Clang |
| License Mode | LGPL-2.1 (no GPL, no non-free) |
| Build Type | Static libs → merged shared library + standalone executables |
| Size Optimization | `--enable-small`, `-Os -fpic` |

## License

This build is compiled with `--disable-gpl --disable-nonfree`, making it a **pure LGPL-2.1** build. You may use it in proprietary / commercial applications as long as you comply with the [LGPL-2.1](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html) license terms.

This repository distributes prebuilt binaries of FFmpeg. The complete FFmpeg source code is available at:

- **Official**: [https://ffmpeg.org/download.html](https://ffmpeg.org/download.html)
- **Git**: [https://git.ffmpeg.org/ffmpeg.git](https://git.ffmpeg.org/ffmpeg.git)
- **GitHub mirror**: [https://github.com/FFmpeg/FFmpeg](https://github.com/FFmpeg/FFmpeg)
- **This build**: tag `n8.0.1`

FFmpeg includes code from many contributors. Full copyright information is available in the FFmpeg source distribution.

---

If this project saved your time, please consider giving it a ⭐ — it helps others find it too!
