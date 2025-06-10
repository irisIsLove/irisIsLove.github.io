---
title: 在 vcpkg 下如何使用 pkg-config
date: 2025-06-10 18:03:54
categories:
- 技术
tags:
- C/C++
- CMake
- vcpkg
---

# 在 vcpkg 下如何使用 pkg-config

笔主本人在用 vcpkg 安装 FFmpeg 的时候发现，安装完成的输出里面有对应库在cmake中的使用方法
```shell
The following packages are already installed:
    ffmpeg[core,x265,x264,gpl,ffprobe,ffplay,ffmpeg,zlib,xml2,webp,vpx,vorbis,theora,tesseract,ssh,srt,speex,soxr,snappy,sdl2,qsv,opus,openmpt,openjpeg,openh264,opengl,opencl,nvcodec,mp3lame,modplug,lzma,ilbc,iconv,fribidi,freetype,fontconfig,dav1d,bzip2,ass,aom,amf,all,swscale,swresample,avformat,avfilter,avdevice,avcodec]:x64-windows@7.1.1#2
Total install time: 4.28 ms
ffmpeg provides CMake integration:

  find_package(FFMPEG REQUIRED)
  target_include_directories(main PRIVATE ${FFMPEG_INCLUDE_DIRS})
  target_link_directories(main PRIVATE ${FFMPEG_LIBRARY_DIRS})
  target_link_libraries(main PRIVATE ${FFMPEG_LIBRARIES})

ffmpeg provides pkg-config modules:

  # FFmpeg codec library
  libavcodec

  # FFmpeg device handling library
  libavdevice

  # FFmpeg audio/video filtering library
  libavfilter

  # FFmpeg container format library
  libavformat

  # FFmpeg utility library
  libavutil

  # FFmpeg audio resampling library
  libswresample

  # FFmpeg image rescaling library
  libswscale

```

使用第一种方法的时候，cmake会检索出所有的 ffmpeg 库，作为强迫症的我不太喜欢这种方式，感觉不优雅，于是发现第二种方法，通过 DeepSeek 的解释大致了解了用法

```cmake
find_package(PkgConfig REQUIRED)
pkg_check_modules(AVCODEC REQUIRED libavcodec)
pkg_check_modules(AVFORMAT REQUIRED libavformat)

target_link_libraries(my_app PRIVATE
    ${AVCODEC_LIBRARIES}
    ${AVFORMAT_LIBRARIES}
)
```

这样就可以精确控制 FFmpeg 的依赖项，而不会引入不必要的库

`ls <vcpkg-root>/installed/<triplet>/lib/pkgconfig/`
使用上述命令，就可以直接查看 pkgconfig/ 目录