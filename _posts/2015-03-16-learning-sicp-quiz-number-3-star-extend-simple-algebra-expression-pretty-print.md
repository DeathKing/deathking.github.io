---
layout: post
title: "Learning-SICP 趣题 #3*. 美观输出扩展的简单代数表达式"
subtitle: "Learning SICP Quiz #3*. Extend Simple Algebra Expression Pretty-Print"
formatted_title: "Learning-SICP 趣题 #3*.<br />美观输出扩展的简单代数表达式"
modified: 2015-03-16 18:11:07 +0800
tags: [sicp,scheme,lisp,functional,programming,hacker,higher order procedure,高阶函数,函数式编程]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
mathjax: y
disqus: y
---

## Meta

+ Quiz难度：★★★★☆
+ 涉及章节：《计算机程序的构造和解释》2.2.4 实例：一个图形语言
+ 涉及知识点：依赖反转 符号计算 分情况分析

## 绪言

在[Quiz 3](/2015/03/13/learning-sicp-quiz-number-3-simple-algebra-expression-pretty-print/)中，我们讨论了如何输出简单的代数表达式。其中主要处理二元的四则运算符，在本次Quiz中，我们将引入一个新的二元运算符$ \; power \; $和几个一元运算符：$ factorial $、$ negtive $ 和 $ sqrt $。

## 简单表达式扩展版本的上下文无关文法

<div>
$$
\begin{align*}
  \left< number \right> \longrightarrow  & \; \text{ any Scheme number } \\
  \left< variable \right> \longrightarrow  & \; \text{ any Scheme symbol }  \\
  \left< constant \right> \longrightarrow  & \; \pi \: |\: e\: |\: \hbar  \\
  \left< atom \right> \longrightarrow  & \; \left< variable \right> |\left< constant \right> |\left< number \right>  \\
  \left< unary-operator \right> \longrightarrow & \; negtive \; | \; factorial \; | \; sqrt \\
  \left< binary-operator \right> \longrightarrow  &  \; + \;| \; - \; | \; \ast \;| \; \; / \; \; | \; power \\
  \left< exp \right> \longrightarrow & \; \left< atom \right> | \\
   & \; ( \left< unary-operator \right> \left< exp \right> ) \; | \\
   &\; ( \left< binary-operator \right> \left< exp \right> \left< exp \right>  )
\end{align*}$$
</div>
