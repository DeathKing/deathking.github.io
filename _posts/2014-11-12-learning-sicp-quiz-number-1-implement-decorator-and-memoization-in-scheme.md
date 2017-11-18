---
layout: post
title: "Learning-SICP 趣题 #1. 用Scheme实现装饰器和记忆化技术"
subtitle: "Learning-SICP Quiz #1. Implement Decorator and Memoization In Scheme"
formatted_title: "Learning-SICP 趣题 #1. </br>用Scheme实现装饰器和记忆化技术"
modified: 2014-11-12 15:16:44 +0800
tags: [scheme, decorator, lisp, sicp, programming, trick, memoization]
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
+ 涉及章节：《计算机程序的构造和解释》1.2节过程与它们产生的计算
+ 涉及知识点：递归 记忆化 词法作用域 高阶函数 装饰器

## 概述

众所周知，Fibonacci 数列是这样定义的，首先是基准条件：

<div>$$ F_0 = F_1 = 1 $$</div>

然后是递推关系：

<div>$$ F_n = F_{n - 1} + F_{n - 2} (n \ge 2) $$</div>

通常来说，编写计算 Fibonacci 数列的 Scheme 代码非常容易：

```scheme
(define (fib n)
  (if ((or (eq? n 1) (eq? n 2)))
    1
    (+ (fib (- n 1)) (fib (- n 2)))))
```

事实上，这段代码的计算效率非常不理想。即使是 `(fib 30)` 这样的调用也会计算相当长的一段时间。正如书中讲解的那样，这是由于上述这段**幼稚**的代码在计算 `(fib 30)` 时产生了大量的重复计算。既然我们知道了瓶颈所在，那么优化起来就非常简单了，一种很容易想到的方法就是**“[记忆（Memoization）](http://en.wikipedia.org/wiki/Memoization)”**。

由于像 `fib()` 这样数学意义上的函数（也就是 Scheme 中所强调的**纯函数**）在计算过程中并不会产生**副作用（Side Effect）**。这样，无论调用几次 `fib(n)` ，得到的结果都应该是相同的。因此，我们可以使用某种数据结构存储 `fib(n)` 的值，在计算 `fib(n+1)` 时即可直接使用我们记住的这个值，无需重复计算。

使用这种“记忆”技术并不会花太大的功夫。在正式开始 Quiz 之前，我们有必要看一点来自 Python 的“异域风情”。在 Python 中，形如 `@deco` 这样以 `@` 开头的符号被称作**“装饰器（Decorator）”**。装饰器不过是一个单参数函数（大多数情况如此，但 Python 中的装饰器还允许接收其它参数）的函数，接受的参数是一个函数，装饰器会在这个函数周围添加一些自定义代码——这可能就是“装饰”一词的来源，但无论如何装饰器最后总是返回一个函数，即“包装后”的函数。

## 要求

为什么不引入一点异域风味呢？考虑到 Scheme 中的 `@` 符号是可以作为标识符的合法词法（MIT-Scheme 允许我们这样做，但 R<sup>5</sup>RS 却认为 `@` 符号应该保留给以后的实现），并且 Scheme 中函数亦是一等公民，我们显然可以在 Scheme 中实现装饰器。对于上面的无记忆版本的`fib`过程来说，我们可以神奇地调用：

```scheme
(@memoize fib)
; Value : #[compound-procedure]
```

现在我们就得到了一个经过被“记忆化”技术装饰过的函数，再经过一些技术手段的处理，我们调用这个被装饰后的函数，即可很快算得计算结果。

需要注意的是：

1. 调用 `(@memoize fib)` 之后，`fib` 函数本身没发生改变；
2. 注意到有些递归函数不一定是单参的，比如著名的 Ackerman 函数，你的`@memoize`装饰器必须能够处理参数大于或等于1的纯函数；
3. 无论你将`@memoize`实现为函数还是语法，请给出如何使用这个装饰器，并使用相应的`profile`函数来做简单的评测（提示：Scheme 用户可以使用 `with-timings` 过程，这是 MIT-Scheme 内建支持的过程）。

## 附录

### Ackerman 函数的 Scheme 代码

```scheme
(define (ack m n)
  (cond
    ((eq? m 0) (+ n 1))
    ((and (eq? n 0) (> m 0)) (ack (- m 1) 1))
    ((and (> n 0) (> m 0)) (ack (- m 1) (ack m (- n 1))))
    (else (error "invalid arguments with" m n))))
```
### 可以用于评测的基准测试代码

```scheme
(load-option 'format)

(define (benchmark describetion exp)
  (with-timings exp
    (lambda (run-time gc-time real-time)
      (format #t "~A: ~As\n" describetion (internal-time/ticks->seconds real-time)))))
```
