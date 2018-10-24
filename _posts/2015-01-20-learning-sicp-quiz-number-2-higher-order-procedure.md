---
layout: post
title: "Learning-SICP 趣题 #2. 高阶过程"
subtitle: "Learning SICP Quiz #2. Higher Order Procedure"
formatted_title: "Learning-SICP 趣题 #2.<br>高阶过程"
modified: 2015-01-20 11:49:36 +0800
tags: [sicp,scheme,lisp,functional,programming,hacker,higher order procedure,高阶函数,函数式编程]
image:
  feature: 
  credit: 
  creditlink: 
comments: y
disqus: y
share: y
mathjax: y
---

## Meta

+ Quiz难度：★★★☆☆
+ 涉及章节：《计算机程序的构造和解释》2.2.3小节 序列作为一种约定的界面
+ 涉及知识点：高阶过程 组合子

## 序章

通常来说，我们可以通过将某些重复的操作实现为高阶过程，来抓取处理数据的公共模式。比如，Scheme 中的 `map` 过程，就有这样的一种思想：

> `(map proc seq)`  
> 将过程 `proc` 应用在表 `seq` 中的每一个元素上，并将应用的结果收集并组合成一个新的表，返回给调用者。

实际上，有一种被称为**归约（Reduce）**的操作，它试图减少操作数的数目。`reduce` 过程可以以 `(reduce op initial sequence)` 的形式调用，它的定义如下：

```scheme
(define (reduce op initial sequence)
  (if (null? sequence)
      initial
      (op (car sequence)
          (reduce op initial (cdr sequence)))))
```

如果我们想计算 `$ 1 + 2 + 3 + ... + 100 $` 的结果，那么我们就可以调用 `(reduce + 0 (iota 100 1))` 
算得正确结果 5050 。其中，[`iota`](https://www.gnu.org/software/mit-scheme/documentation/mit-scheme-ref/Construction-of-Lists.html) 是 MIT-Scheme 提供的一个过程，`(iota end begin)` 可以生产从 `begin` 开始，`end` 结束，步进为1的表。

现在，你遇到了第一个挑战：

1. 请分析 `reduce` 的定义，指明它是**右结合(right-association)**的，亦即，对于 `(reduce - 0 (list 1 2 3))` ，它的实际运算顺序是：`$ (1 - (2 - (3 - 0))) $`。
2. 实际上，我们日常的四则运算都是**左结合(left-association)**，也就是实际运算顺序是`$ (((0 - 1) - 2) - 3) $`，请在理解练习1的基础上，重新定义 `reduce-left`，使它以左结合的方式运算序列。

## 用reduce来定义其它过程

SICP 中强调了一门好的语言应该有三个基本机制：

1. **取用**：使用基本原语的机制
2. **组合**：将基本原语组合成复合结构的机制
3. **抽象**：将复合结构抽象为基本原语的机制

一旦定义好 `reduce` 操作后，我们就可以认为它是一个基本原语，并用它来构造其它的函数。第二个挑战出自SICP书中的练习2.33，请尝试用填写下面表达式中缺失的部分：

```scheme
(define (map p sequence)
  (reduce (lambda (x y) <??>) nil sequence))

(define (append seq1 seq2)
  (reduce cons <??> <??>))

(define (length sequence)
  (reduce <??> 0 sequence))
```

如果读者不太熟悉 `map` 、`append` 和 `length` 的语义，可以查询MIT-Sceheme的参考手册。

## 实现reduce-operator

在Scheme中，我们可以认为`(list 1 2 3)`等价于`(cons 1 (cons 2 (cons 3 '())))`。实际上，在Lisp或者函数式编程中，很多时候都有类似的需求：某个过程`proc*`是以“归约”的形式调用`proc`，这样，一旦定义好`proc`后，用户不需要再额外给出`proc*`的定义，而只是简单地加一句声明或者是复合，比如：

```scheme
(define list (reduce-operator cons '() 'right))
; list是一个由cons以reduce形式构成而成的过程
; 在这个操作中，空值（初始值）是空表'()
; 'right表示这个归约过程是右结合的

; MIT-Scheme中允许用户以declare语句来实现这个功能
(declare (reduce-operator (list cons (null-value '() any) (group right))))
; list是一个由cons以reduce形式构成而成的过程
; 在这个操作中，空值（初始值）是空表'()
; any则允许list过程有任意多个参数
; (group right)表示list是右结合的
```

你的第三个挑战，则是实现`reduce-operator`，你可以实现为上述代码的第一个风格，也可以实现为MIT-Scheme中的风格。实现MIT-Scheme中的风格会稍微困难一点，需要用到宏，这给读者需要仔细思考以解决这个问题。