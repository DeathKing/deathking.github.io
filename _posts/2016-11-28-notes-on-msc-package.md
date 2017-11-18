---
layout: post
title: "MSC 宏包使用注记"
subtitle: "Notes on MSC Package"
modified: 2016-11-28 06:44:20 +0800
tags: [msc, macro, latex, writting, crypto]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
disqus: y
---

MSC（Message Sequence Chart）宏包提供了一套称为 MSC 的语言，用于绘制不同系统组件之间的交互图像。在绘制协议图像方面尤为有用。MSC包本身的使用问题可以参见宏包附带的参考手册，这里整理一些使用本宏包时需要注意的一些问题。

## 编译相关

推荐使用 XeLaTeX 编译文档，特别是在 beamer 中使用本宏包时尤为需要注意。

## 移除生成图表中的 MSC 字样

参考资料：[pdftex - How to get rid of "msc" text on image using LaTeX and Tikz (PsTricks) package?](http://tex.stackexchange.com/questions/335354/how-to-get-rid-of-msc-text-on-image-using-latex-and-tikz-pstricks-package)

```latex
\renewcommand\msckeyword{} 
\renewcommand\hmsckeyword{}
\renewcommand\mscdockeyword{}
```

## 移除生成图表的边框

在 `msc` 环境中使用 `\drawframe{no}` 命令去掉边框绘制。

```ruby
\begin{msc}{}
\drawframe{no}
\end{msc}
```

## 编排多行的 `action`

使用 `\action*` 命令和 `\parbox`，使用 `\action*` 命令使得 MSC 可以自行计算正确的行高。

```ruby
\action*{\parbox{6.4\instwidth}{\centering Computes {\color{red} $ex'_c, dn'$} s.t. $\hash(log^c_1)=\hash(log^s_1)$ \\
by finding a \textbf{chosen-prefix collision} {\color{red} $(C_1, C_2)$} s.t.:  \\
$\hash(\texttt{CH} \mid {\color{red} \texttt{SH}' \mid \texttt{SC}' \mid \texttt{SKE}' \mid \texttt{SCR}'(C_1 \mid -)}) = \hash({\color{red} \texttt{CH}'(n_c, C_2)})$}}{m}
```

需要注意的是，如果编排多行的 `\action` ， 在使用 `\nextlevel` 时，需要根据编排的行数，自行指定一个合适的数值。
