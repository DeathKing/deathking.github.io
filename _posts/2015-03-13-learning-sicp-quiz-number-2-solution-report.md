---
layout: post
title: "Learning-SICP 趣题 #2. 解题报告"
subtitle: "Learning-SICP Quiz #2. Solution Report"
formatted_title: "Learning-SICP 趣题 #2.<br />解题报告"
modified: 2015-03-13 00:03:22 +0800
tags: [sicp,scheme,lisp,functional,programming,hacker,higher order procedure,高阶函数,函数式编程]
image:
  feature: 
  credit: 
  creditlink: 
comments: y
disqus: y
share: y
---

## 序章

```scheme
(define (reduce op initial sequence)
  (if (null? sequence)
      initial
      (reduce op (op initial (car sequence)) (cdr sequence))))
```
