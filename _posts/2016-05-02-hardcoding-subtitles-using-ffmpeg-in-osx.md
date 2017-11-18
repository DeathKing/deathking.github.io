---
layout: post
title: "在 Mac OSX 系统中使用 FFMPEG 为影片添加硬字幕"
subtitle: "Hardcoding subtitles using ffmpeg in OSX"
modified: 2016-05-02 20:26:05 +0800
tags: [subtitles, ffmpeg, hardcoding, 硬字幕, osx]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
disqus: y
---

首先需要安装 `ffmpeg` ，推荐使用 Homebrew 安装，注意在安装时需要添加一些额外的编译选项（主要是为了启用 `libass`）：

```ruby
brew install ffmpeg --with-fdk-aac --with-ffplay --with-freetype --with-libass --with-libquvi --with-libvorbis --with-libvpx --with-opus --with-x265
```

接着，调用 `ffmpeg` ，命令为：

```ruby
ffmpeg -i video.avi -vf subtitles=subtitle.srt out.avi
```

其中，video.avi 是视频输入源，subtitle.srt 是字幕（支持ass格式），out.avi是视频输出。

这样，我们就可以很方便地为视频添加硬字幕，不需要在去费心地找相关软件了。

## 参考链接

+ [1] [CompilationGuide/MacOSX – FFmpeg](https://trac.ffmpeg.org/wiki/CompilationGuide/MacOSX)
+ [2] [HowToBurnSubtitlesIntoVideo – FFmpeg](https://trac.ffmpeg.org/wiki/HowToBurnSubtitlesIntoVideo) 