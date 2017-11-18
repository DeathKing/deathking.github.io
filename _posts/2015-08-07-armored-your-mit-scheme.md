---
layout: post
title: "武装你的 MIT-Scheme"
subtitle: "Armored Your MIT-Scheme"
modified: 2015-08-07 14:06:43 +0800
tags: [mit-scheme, enhance, configuration, tools]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: y
disqus: y
---

“MIT-Scheme 太难用了！”我常常听到 SICP 的学习者这样跟我抱怨。

由于年代久远也缺乏更新，MIT-Scheme 确实不如一些新进的 Scheme 实现好用。自己动手，丰衣足食，虽然我们不能直接改造 MIT-Scheme 的实现，但我们可以为 MIT-Scheme 添砖加瓦，让它变得好用。

## 添加 rlwrap

MIT-Scheme 不支持方向键导航，如果我们发现前面有个字符输错了，就得删掉后面已经输入的内容，重新输入整个命令。同时，MIT-Scheme 并没有提供记忆历史输入的功能，我们不能通过上下键在多次输入命令间移动，这也非常不爽。

<img src="/images/post/mit-scheme-no-nav.png" alt="" class="has-shadow img-margin display" style="width: 85%;">


`rlwrap` 支持**历史记录**、**Tab键补全**、**方向键导航**甚至**括号匹配**等功能，可以用来提升 MIT-Scheme 的使用体验。

我们首先需要安装 `rlwrap` ，安装了 Homebrew 的用户可以执行下面的命令安装：

```ruby
brew install rlwrap
```

你还需要将这个 [gist](https://gist.github.com/bobbyno/3325982) 保存成一个文件放在你的主目录下，取名叫 `.scheme_completion.txt` 是个不错的选择。安装完成后，可以按照下面的命令启动 Scheme ：

```ruby
rlwrap -r -c -f "$HOME"/.scheme_completion.txt scheme
```

当然，如果读者觉得每次都输这么长的命令很麻烦，那么定义一个 Shell 函数封装这个命令也是不错的选择。

<img src="/images/post/mit-scheme-complete.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

## 安装十全大补丸：SLIB

SLIB 是由 Aubrey Jaffer 开发的一套可移植 Scheme 程序设计语言函数库。它支持多种 Scheme 实现、功能齐全、文档丰富，是您居家旅行，杀人越货的不二选择。

Windows 和 Linux 系统都有相应的安装包，这里介绍如何使用源码安装。首先访问 [SLIB主页](http://people.csail.mit.edu/jaffer/SLIB.html) 下载 SLIB 源码，并解压。解压完毕后，将文件拷贝到 `/usr/local/lib/slib` 目录下。使用 `ls` 命令查看该目录下的结构，你应该获得下面这样的结果：

<img src="/images/post/mit-scheme-slib.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

使用 `cd` 命令切换到这个目录下，执行命令安装文档：

```ruby
make infoz
make install
```

在这个文件夹下，有一个名为 `mitscheme.init` 的文件，它的内容大致如下：

<img src="/images/post/mit-scheme-slib.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

将里面的内容全部复制到 `~/.scheme.init` 文件中，再次启动 Scheme ，如果发现提示加载了 `require.scm` 文件，这就说明 SLIB 已成功安装。

<img src="/images/post/mit-scheme-slib-succ.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

## 欢迎来稿！

如果您有关于 MIT-Scheme 的更多使用技巧，请让我知道！

## 参考资料

+ [mit-scheme REPL with command line history and tab completion](http://stackoverflow.com/questions/11908746/mit-scheme-repl-with-command-line-history-and-tab-completion)
+ [The SLIB Portable Scheme Library](http://people.csail.mit.edu/jaffer/SLIB)
