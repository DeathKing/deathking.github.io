---
layout: post
title: "Learning-SICP 趣题 #4. 解析代数表达式为S-表达式"
subtitle: "Learning-SICP Quiz #4. Parsing Algebra Expression to S-Expression"
formatted_title: "Learning-SICP 趣题 #4.<br />解析代数表达式为S-表达式"
modified: 2015-03-20 13:00:09 +0800
tags:  [scheme, lisp, sicp, programming, parsing, sexp]
image:
  feature: 
  credit: 
  creditlink: 
comments: y
disqus: y
mathjax: y
---

## Meta

+ Quiz难度：★★★☆☆
+ 涉及章节：无
+ 涉及知识点：词法分析 语法分析

## 绪言

在 [Quiz 3](2015/03/13/learning-sicp-quiz-number-3-simple-algebra-expression-pretty-print/) 和 [Quiz 3*](/2015/03/16/learning-sicp-quiz-number-3-star-extend-simple-algebra-expression-pretty-print) 中，我们讨论如何美观地输出以S-表达式表示的代数表达式。S-表达式是一种“前缀记法”，也称作“波兰表示法”，即把运算符（Operator）放到最前面，其后才是由运算对象（Operand）组成的序列。然而在我们的日常生活中，更习见的却是“中缀记法”，即对于二元操作，其操作符通常置于两操作数中间。

因此，为了能够使用我们在 Quiz 3 开发的程序，我们有必要开发另一个程序，能将中缀表示的代数表达式转换为能被我们程序所处理的S-表达式。

## 代数表达式的文法

<div>

$$

\begin{align*}
  \left< char   \right> & \longrightarrow & \; a \; | \; b \; | \; \dots \; | \; z \\
  \left< digit  \right> & \longrightarrow & \; 0 \; | \; 1 \; | \; 2 \; | \; \dots \; | \; 9 \\
  \left< sign   \right> & \longrightarrow & \; \epsilon \; | \; + \; | \; - \\
  \left< number \right> & \longrightarrow & \; \left< digit \right> ^+ \\
  \left< variable \right> & \longrightarrow & \; \left< char \right> ^+ \\
  \left< decimal \right>  & \longrightarrow & \; \left< sign \right> \left< number \right> \; |
                                           \; \left< sign \right> \left< number \right>  . \left< number \right> \\
  \left< term   \right>   &\longrightarrow & \; \left< term \right> \; | \; \left< sign \right> \left< term \right> \; | \; ( \left< exp \right> ) \; | \; \left< decimal \right> \; | \; \left< variable \right> \\

  \left< exp_{\Pi} \right> & \longrightarrow & \; \left< term \right> \; | \; \left< exp_{\Pi} \right> \ast \left< term \right> \; | \; \left< exp_{\Pi} \right> / \left< term \right> \; \\
  \left< exp_{\Sigma} \right> &\longrightarrow & \; \left< exp_{\Sigma} \right> + \left< exp_{\Pi} \right> \; | 
  \; \left< exp_{\Sigma} \right>  - \left< \exp_{\Pi} \right> \\

  \left< exp \right> & \longrightarrow & \; \left< term \right> \; |
                                       \; \left< exp \right> +    \left< term \right>  \; |
                                       \; \left< exp \right> -    \left< term \right>  \; | \\

                    &               & \; \left< exp \right> \ast \left< term \right>  \; |
                                       \; \left< exp \right> /    \left< term \right>  \; |
                                       \; \left< exp \right> \text{^} \left< exp \right> \; |
                                       \; SQRT(\left< exp \right>) \; |

\end{align*}

$$
</div>

## 代数表达式的语法规则
