---
layout: post
title: "Brief Introduction to Formal Semantics"
modified: 2015-05-04 00:01:12 +0800
tags: []
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
mathjax: y
---



## 指称语义

+ 重要人物：
	+ Christopher Strachey
	+ Dana Stewart Scott
	+ Michael Smyth：引入幂域（Power Domains）
	+ Gordon Plotkin：
	+ William D. Clinger：贡献了并发计算Actor模型的指称语义 
+ 基本理论：域理论（Domain Theory）、不动点理论（Fixed-point Theory）
+ 基本思想：通过语义解释函数，用数学对象来注释程序设计语言

# 语义解释函数、论域方程

<div>
$$
\begin{align}

 \mathscr{C} ⟦e_1 := e_2⟧ \rho \, \sigma = & \, update(\alpha , \beta) \rho \\
	            & \underline{where} \, \alpha = \mathscr{E} ⟦e_1⟧ \rho \, \sigma \, \underline{onto} \, L \\
	            & \underline{and} \, \beta = \mathscr{E} ⟦e_2⟧ \rho \, \sigma \, \underline{onto} \, V

\end{align}
$$
</div>