# Android FFmpeg Prebuilt

[![GitHub stars](https://img.shields.io/github/stars/hzw1199/Android-FFmpeg-Prebuilt?style=social)](https://github.com/hzw1199/Android-FFmpeg-Prebuilt)
[![FFmpeg](https://img.shields.io/badge/FFmpeg-8.1.1%20%7C%208.0.1-green)](https://ffmpeg.org/)
[![Platform](https://img.shields.io/badge/Platform-Android-brightgreen)](https://developer.android.com/)
[![ABI](https://img.shields.io/badge/ABI-arm64--v8a-blue)]()
[![License](https://img.shields.io/badge/License-LGPL--2.1-orange)](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html)

Android 平台 FFmpeg 预编译二进制文件 — 开箱即用，无需编译。

提供 `ffmpeg` / `ffprobe` 可执行文件，以及将所有核心模块合并为单个 `libffmpeg.so` 共享库。

---

## 特性

- **FFmpeg 8.1.1 & 8.0.1** — 提供两个版本的预编译产物，按需选用
- **Android arm64-v8a (aarch64)** — 面向现代 64 位设备，minSdkVersion 28
- **单一共享库** — `libffmpeg.so` 将 libavcodec、libavformat、libavfilter、libavutil、libavdevice、libswresample、libswscale 全部 7 个核心库合并为一个 `.so`
- **独立可执行文件** — `ffmpeg` 和 `ffprobe` 可直接推送到设备通过 shell 运行
- **包含 C/C++ 头文件** — 可直接集成到 Android NDK 项目
- **纯 LGPL 构建** — `--disable-gpl --disable-nonfree`，可安全用于商业项目
- **MediaCodec 硬件加速** — 通过 Android MediaCodec 实现 H.264 / HEVC / MPEG-4 硬件解码
- **16 KB 页面大小兼容** — 符合 [Google Play 16 KB 页面大小要求](https://developer.android.com/guide/practices/page-sizes)，可用于面向 Android 15+ 的应用上架
- **体积优化** — 使用 `--enable-small -Os` 编译

## 文件结构

两个版本的目录结构完全一致，仅以 `ffmpeg-8.1.1/` 为例：

```
ffmpeg-8.1.1/
├── bin/
│   ├── ffmpeg           # 命令行可执行文件 (15 MB)
│   └── ffprobe          # 命令行可执行文件 (15 MB)
├── libffmpeg.so         # 合并共享库 — 全部 7 个模块合一 (75 MB，未 strip)
├── include/
│   ├── libavcodec/
│   ├── libavdevice/
│   ├── libavfilter/
│   ├── libavformat/
│   ├── libavutil/
│   ├── libswresample/
│   └── libswscale/
└── examples/            # FFmpeg 官方示例源码
```

> 8.0.1 版本位于 `ffmpeg-8.0.1/`，文件大小近似相同。

### 关于 libffmpeg.so 体积（strip）

为方便调试，仓库中提供的 `libffmpeg.so` 保留了完整符号信息，体积接近 80 MB。如需用于生产/发布，可使用 NDK 自带的 `llvm-strip` 手动剥离符号，体积可缩小到 **约 15 MB**：

```bash
# 以 NDK r28 为例
$ANDROID_NDK_HOME/toolchains/llvm/prebuilt/<host-tag>/bin/llvm-strip \
    --strip-unneeded \
    ffmpeg-8.1.1/libffmpeg.so
```

其中 `<host-tag>` 在 macOS 上为 `darwin-x86_64`，Linux 上为 `linux-x86_64`，Windows 上为 `windows-x86_64`。

## 编译配置

### 编码器与解码器

已启用所有 LGPL 兼容的编码器和解码器，包括 MediaCodec 硬件解码器：

| MediaCodec 解码器 | 说明                                         |
| ----------------- | -------------------------------------------- |
| h264_mediacodec   | 通过 Android MediaCodec 进行 H.264 硬件解码  |
| hevc_mediacodec   | 通过 Android MediaCodec 进行 HEVC 硬件解码   |
| mpeg4_mediacodec  | 通过 Android MediaCodec 进行 MPEG-4 硬件解码 |

### 其他能力

| 类别                | 状态                    |
| ------------------- | ----------------------- |
| 封装器 / 解封装器   | 全部启用                |
| 协议                | 全部启用（含网络协议）  |
| 滤镜                | 全部启用                |
| 解析器 / 码流过滤器 | 全部启用                |
| 硬件加速            | MediaCodec + JNI 已启用 |
| iconv               | 已启用                  |

### libffmpeg.so 包含的库

`libffmpeg.so` 由以下全部 7 个静态库链接而成：

| 库            | 说明                     |
| ------------- | ------------------------ |
| libavdevice   | 设备采集与播放           |
| libavfilter   | 音视频滤镜框架           |
| libavcodec    | 音视频编解码             |
| libavformat   | 封装与解封装（容器格式） |
| libswresample | 音频重采样与格式转换     |
| libswscale    | 视频缩放与像素格式转换   |
| libavutil     | 通用工具函数             |

## 快速开始

### 作为命令行工具在设备上使用

```bash
adb push ffmpeg-8.1.1/bin/ffmpeg /data/local/tmp/
adb shell chmod +x /data/local/tmp/ffmpeg
adb shell /data/local/tmp/ffmpeg -version
```

### 在 Android NDK 项目中使用 libffmpeg.so

**CMakeLists.txt**

```cmake
add_library(ffmpeg SHARED IMPORTED)
set_target_properties(ffmpeg PROPERTIES
    IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/libs/${ANDROID_ABI}/libffmpeg.so
)
target_include_directories(your_target PRIVATE ${CMAKE_SOURCE_DIR}/ffmpeg-8.1.1/include)
target_link_libraries(your_target ffmpeg)
```

## 编译信息

| 项目          | 值                                    |
| ------------- | ------------------------------------- |
| FFmpeg 版本   | 8.1.1、8.0.1                          |
| 目标平台      | Android                               |
| 目标 ABI      | arm64-v8a (aarch64)                   |
| minSdkVersion | 28 (Android 9.0)                      |
| NDK 版本      | r28                                   |
| 工具链        | LLVM / Clang                          |
| 许可证模式    | LGPL-2.1（无 GPL、无 non-free）       |
| 构建方式      | 静态库 → 合并共享库 + 独立可执行文件 |
| 体积优化      | `--enable-small`、`-Os -fpic`     |

## 许可证

本构建使用 `--disable-gpl --disable-nonfree` 编译，属于**纯 LGPL-2.1** 构建。您可以在专有/商业应用中使用，只需遵守 [LGPL-2.1](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html) 许可证条款。

本仓库分发 FFmpeg 的预编译二进制文件。完整的 FFmpeg 源代码可从以下地址获取：

- **官方**: [https://ffmpeg.org/download.html](https://ffmpeg.org/download.html)
- **Git**: [https://git.ffmpeg.org/ffmpeg.git](https://git.ffmpeg.org/ffmpeg.git)
- **GitHub 镜像**: [https://github.com/FFmpeg/FFmpeg](https://github.com/FFmpeg/FFmpeg)
- **本次构建对应**: tag `n8.1.1`、`n8.0.1`

FFmpeg 包含众多贡献者的代码。完整的版权信息请参阅 FFmpeg 源代码发行版。

---

如果这个项目帮到了你，请考虑给一个 ⭐ — 也能帮助更多人发现它！
