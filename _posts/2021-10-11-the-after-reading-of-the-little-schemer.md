---
layout: post
title: "《小“阴谋家”》注记之CPS变换"
subtitle: "The after reading of The Little Schemer Book (I) -- CPS Transformation"
modified: 2021-10-11 20:04:33 +0800
tags: [scheme, compiler, programming, lisp, y combinator, CPS]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share:
mathjax: y
disqus: y
---

说来惭愧，学习 Scheme 多年，也是最近才把经典书籍 *The Little Schemer* 给读完了。

作为一本入门书籍，本书旨在教会大家如何“用递归的思维方式解决问题”。如果读者具有一定的 Scheme/Lisp 基础，那么完全可以在一天内读完。本书的前几个章节比较 trivial，在引入 Scheme 的基本运算与表结构后，Dan Friedman 向大家介绍了什么是自然递归，以及如何通过对表或自然数的自然递归解决问题，同时也讲解了线性递归、树形递归等不同“形状”的递归。

老书新读，温故知新，这本书的珠玑全在细微之处。例如，第8章介绍了Continuation-Passing-Style 的程序设计方式，第9章引导大家推导 Y 组合子，第10章介绍了求值器的原理，其中任何一个章节都能够引入一个宏大的主题。由于我购买的是中文版，在阅读过程中，发现某些地方译者似乎都没能完全明白 Dan 的本意，因此想在这里同大家分享一下我的思考。

## Conitnuation 与 CPS 风格初感受

首先请让我们思考一下 Scheme 的求值规则，为了求值表达式：

```scheme
(display (+ 1 2))
```

Scheme 需要先求值子表达式 `(+ 1 2)` ，程序控制权转移到子表达式内部，对于 `display` 语句来说：

```scheme
(display <retval>)
```

它期待着子表达式返回一个值 `<retval>`，以完成后续的过程调用。这个动作隐式地存在于解释器的求值规则中，现在，“显式”地把它变成程序自身的行为：

```scheme
(lambda (retval) (display retval))
```

这里，我们通过一个 `lambda` 抽象创建了一个只接收单个值的继续（continuation）。如果我们有一个 CPS 风格的 `+cps` 函数，那么就可以向下面这样调用：

```scheme
(+cps 1 2 (lambda (retval) (display retval)))
```

在 `+cps` 内部求值完 `1+2` 后，会将得到的值作为参数去调用该继续，所以有时在其它程序语言社区，继续也被叫做回调函数（callback function）。由于 `cps+` 的实现要依赖一个由解释器提供的非 CPS 风格的加法，因此代码也要依赖于“函数返回”的模型。我们的实现通过显式地使用 `(k sum)` ，指出 CPS 风格的程序都是通过函数调用实现“返回”的 ：

```scheme
(define (+cps a b k)
  (let ((sum (+ a b)))
    (k sum)))
```

在 CPS 风格中，“返回”这个行为只是函数调用的语法糖衣，调用者（caller）与被调用者（callee）的角色发生了，或者说，消除了传统意义上的“调用者”。

简单总结一下：

1. **继续**是一个有单个或多个参数的过程；
2. **继续传递风格**中的函数一般带有一个额外参数 `k` 来接收传递而来的继续，并通过在尾表达式处调用该函数，实现控制权转移。

## 如何运行 CPS 风格的函数？

为了在常见的解释器中运行 CPS 风格的函数，我们通常会定义一个 `id` 函数：

```scheme
(define id (lambda x x))
```

这样，在运行 CPS 风格的函数 `fn-cps` 时，在顶层将 `id` 函数作为继续传递即可。

## 如何将普通函数改写为 CPS 风格？

我们以原书中的 `evens-only*` 函数为例。给定一张表，该函数会构造一张只保留了偶数元素的新表。函数名的 `*` 尾缀暗示了这是一个树形递归，即，如果我们处理的某元素也是一张表，那么就以该元素为参数，递归地调用 `evens-only*` 函数。为了方便阅读，本文采用的代码都以 R^6RS 报告允许的方式改写了括号，读者可以发现 `cond` 形式的子句都以 `[]` 包裹。为了在只支持 R^5RS 的实现上运行代码，读者需要自行对代码做出修改。 

```scheme
(define evens-only*
  (lambda (l)
    (cond
      [(null? l) '()]
      [(atom? (car l))
       (cond
         [(even? (car l))
          (cons (car l) (evens-only* (cdr l)))]
         [else
          (evens-only* (cdr l))])]
      [else
       (cons (evens-only* (car l))
             (evens-only* (cdr l)))])))

;;; (evens-only* '((9 1 2 8) 3 10 ((9 9) 7 6) 2))
;;; => ((2 8) 10 (() 6) 2)
```

那么，如何将 `evens-only*` 改写为 CPS 风格的函数呢？首先我们知道，CPS 风格的函数会多一个额外的参数 `co` ，书上把它叫做 collector ，但更常用的名字是 `k` ，来强调他是一个 `kontinuation`。因此，首先改写 `evens-only*&co` 函数的签名如下：

<div style="margin-bottom: 20px">
<table style="margin:auto">
<tbody>
<tr>
<td>
<code><pre>
(define evens-only*
  (lambda (l)
    (cond
</pre></code>
</td>
<td>
<code><pre>
(define evens-only*&co
  (lambda (l col)
    (cond
</pre></code>
</td>
</tr>
<tr>
<td>正常风格</td>
<td>Continuation-Passing-Style</td>
</tr>
</tbody>
</table>
</div>

此时，`evens-only*&co` 函数的语义会发生细微的变化：给定一张表 `l`，该函数会构造一张只保留了偶数元素的新表 `l^`，并调用 `(co l^)`，没有对函数本身的返回值做出任何承诺。我们需要注意这其中的细微变化。

先考虑其中的原子情况，作为递归的边界条件，当 `l` 为空的时候，我们应该做什么呢？这需要我们考察原函数的语义。`(evens-only* '())` 返回空表，因此，`evens-only*&co` 也应该“返回”空表。在这一层，我们没有需要再进行的计算了，我们以空表为参数调用继续：

<div style="margin-bottom: 20px">
<table style="margin:auto">
<tbody>
<tr>
<td>
<code><pre>
      [(null? l) '()]
</pre></code>
</td>
<td>
<code><pre>
      [(null? l) (co '())]
</pre></code>
</td>
</tr>
<tr>
<td>正常风格</td>
<td>Continuation-Passing-Style</td>
</tr>
</tbody>
</table>
</div>

继续考虑其他情况，当 `(car l)` 为原子时，我么需要根据其奇偶性来决定后续操作。我们先考虑较为简单的 `else` 子句。如果 `(car l)` 是奇数，那么 `(evens-only* l)` 与 `(evens-only* (cdr l))` 的结果应该一致，也就是说，把求解关于 `l` 的子问题，归约为了规模较小一点的，关于 `(cdr l)` 的问题。

```scheme
;;; evens-only*
[(atom? (car l))
  (cond
    [(even? (car l))
     (cons (car l) (evens-only* (cdr l)))]
    [else
     (evens-only* (cdr l))])]  ;;; <== watch here
```

改写只是改变过程“返回结果的方式”，并不改变语义。因此不难想到，改写的过程，也需要递归地调用 `(evens-only*&co <s> <k>)` 过程。

对于 `<s>` ，很容易想到就是 `(cdr l)`。而第二个参数 `<k>`，也就是我们要传递的继续，又应该是什么呢？

子问题 `(cdr l)` 的解，就是关于 `l` 的解，因此对于解的结果。我们不需要任何处理，直接送给“外层”继续：

```scheme
(evens-only*&co (cdr l) (lambda (l^) (co l^)))

;;; 注意到这里构造的继续只是辅助性地传递了“返回值”
;;; 因此可以直接将上层继续传递给子问题
(evens-only*&co (cdr l) co)
```

我们似乎找到了点感觉，大概是什么样的呢：

1. **（递归调用部分）**将本层的问题，转换为一个或多个子问题求解；
2. **（构造继续部分）**将如何关于处理得到的解，形成本层问题的解的过程，封装在继续里，传递给子问题；

因此，请考虑 `(car l)` 为偶数时的情况：

```scheme
;;; evens-only*
[(atom? (car l))
  (cond
    [(even? (car l))  ;;; <== watch here
     (cons (car l) (evens-only* (cdr l)))]
    [else
     (evens-only* (cdr l))])]
```

关于 `l` 的解，是把 `(car l)` 关于子问题的解给 `cons` 起来，因此我们认识到：① 子问题可以调用 `(evens-only*&co (cdr l))` 求解；②
如果子问题求解的结果是 `l^`，那么 `(cons (car l) l^)` 就是最终的答案；③ 可以通过 `(co ...)` 来返回最终答案。因此，我们可以整合得到下面的代码：

```scheme
[(even? (car l))
 (evens-only*&co (cdr l)
  (lambda (l^)                      ; l^ 是子问题的解
    (let [(l^^ (cons (car l) l^))]
      (col l^^))))]
```

掌握窍门后，读者可以轻易地将原过程外层 `cond` 形式的 `else` 子句变换为 CPS 风格，这里不再做赘述。注意到该 `else` 子句的值依赖于两个子问题，如何组织他们的继续是一个值得思考的问题。

有答案了么？下面是原过程与改写为CPS风格后的过程对比。其中，<span style="color:blue">蓝色</span>、<span style="color:orange">橙色</span>、<span style="color:green">绿色</span>表示相应子问题的求解，而<span style="color:red">红色</span>则代表最终解是如何通过子问题构造而来的。读者可以观察到，在 `evens-only*&co` 中，子问题的求解与最终解的构造被清晰地分离开了。

<div style="margin-bottom: 20px">
<table style="margin:auto">
<tbody>
<tr>
<td>
<code><pre>
01. (define evens-only*
02.   (lambda (l)
03.     (cond
04.       [(null? l) <span style="color:red">'()</span>]
05.       [(atom? (car l))
06.        (cond
07.          [(even? (car l))
08.            <span style="color:red">(cons (car l)</span> <span style="color:blue">(evens-only* (cdr l))</span><span style="color:red">)</span>]
09.
10.
11.          [else
12.           <span style="color:red">(</span><span style="color:blue">evens-only* (cdr l)</span><span style="color:red">)</span>])]
13. 
14. 
15.       [else
16.        <span style="color:red">(cons</span> <span style="color:green">(evens-only* (car l))</span>
17.              <span style="color:orange">(evens-only* (cdr l))</span><span style="color:red">)</span>])))
18.
19.
20.
</pre></code>
</td>
<td>
<code><pre>
01. (define evens-only*&co
02.   (lambda (l co)
03.     (cond
04.       [(null? l) <span style="color:red">(co '())</span>]
05.       [(atom? (car l)
06.        (cond
07.          [(even? (car l))
08.           (<span style="color:blue">evens-only*&co (cdr l)</span>
09.             (lambda (<span style="color:blue">l^</span>)
10.               <span style="color:red">(co (cons (car l) l^)))</span>))]
11.          [else
12.           (<span style="color:blue">evens-only*&co (cdr l)</span>
13.             (lambda (<span style="color:blue">l^</span>)
14.               <span style="color:red">(co l^)</span>))])]
15.      [else
16.        (<span style="color:green">evens-only*&co (car l)</span>
17.          (lambda (<span style="color:green">l^</span>)
18.            (<span style="color:orange">evens-only*&co (cdr l)</span>
19.              (lambda (<span style="color:orange">l^^</span>)
20.                <span style="color:red">(co (cons</span> <span style="color:green">l^</span> <span style="color:orange">l^^</span>)))))))])))
</pre></code>
</td>
</tr>
<tr style="text-align:center">
<td><b>正常风格</b></td>
<td><b>Continuation-Passing-Style</b></td>
</tr>
</tbody>
</table>
</div>

## 有任何好处么？

仔细观察 CPS 风格的函数，请注意它的尾表达式（参见R5RS关于尾表达式的描述）：

```scheme
01. (define evens-only*&co
02.   (lambda (l co)
03.     (cond
04.       [(null? l) (co '())]
...
07.          [(even? (car l))
08.           (evens-only*&co (cdr l)
...
11.          [else
12.           (evens-only*&co (cdr l)
...
15.      [else
16.        (evens-only*&co (car l)
```

所有的尾表达式都是一个过程调用，并且除了充当边界条件的第 4 行的调用外，其余的都是自调用，因此这是一个尾递归（tail recursion）。这样，由于 CPS 风格的函数都不需要“返回”，因此在尾递归调用中，没有必要保留调用栈，理论上来说，可以在常量空间的栈上，实现任意数量的尾递归调用。可以参见 [SICP 《Lec1b：计算过程》](https://www.bilibili.com/video/av8515129?p=2) 中解释为什么递归过程也可以产生迭代计算的描述。

通过改写为 CPS 风格的程序，我们真的就能消除掉像 `evens-only*` 那样的函数的树形递归结构吗？当然，没有什么是免费的，我们通过 CPS 变换将原函数变成了尾递归，但跟随尾递归调用而构造出的继续却不断地变得复杂！这些继续通常都是一个闭包，一方面它们通常在堆上分配，另一方面它们还需要捕获定义它们时的环境。对栈的节省，又得靠对堆的消耗来弥补！

也难怪不得有人说：

> CPS stands for Continuation is Poor man’s Stack.

## 后记

The Little Schemer 的中译版制作非常精良，译者翻译也十分用心。在阅读时，我注意到第 142 页的一条脚注：

> 译者注：原文为 “...and that multiinsertLR will return () when lat is empty.” 有误。正确的应该是 “multiinsertLR&co will return ()”。

我能理解译者的意图。但事实上，Dan Friedman 的原文是正确的，同时也是理解 CPS 变换的关键点。也就是说，对于 CPS 变换，我们并不改变函数的“语义”，只是改变了函数的“返回”方式。`multiinsertLR&co` 的定义当然依赖于原函数。

本文仅做为抛砖引玉之介绍，关于 continuation 和 CPS 还有很多有意思的高级主题，希望之后我们能涵盖到。

## 参考资料

关于 CPS 有大量高质量的论文与文章，这里列出其中一小部分：

1. Matt Might, By example: Continuation-passing style in JavaScript, https://matt.might.net/articles/by-example-continuation-passing-style/
2. Tao He, Continuation Passing Style, https://sighingnow.github.io/%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80/continuation_passing_style.html