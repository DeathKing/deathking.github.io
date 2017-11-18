---
layout: post
title: "Learning-SICP 趣题 #1. 解题报告"
subtitle: "Learning-SICP Quiz #1. Solution Report"
modified: 2014-11-14 14:45:33 +0800
tags: [sicp, scheme, decorator, lexcial closure, lisp]
image:
  feature: 
  credit: 
  creditlink: 
comments: y
disqus: y 
---


在[Quiz 1. 用Scheme实现装饰器和记忆化技术](/2014/11/12/learning-sicp-quiz-number-1-implement-decorator-and-memoization-in-scheme/)中，我们要求各位实现一个为函数加上记忆化技术的装饰器，这是一个非常有意思的问题。实际上，类似问题已经在《计算机程序的构造和解释》第三章：模块化、对象和状态中讨论过，并作为练习3.27提供给读者思考。在这篇文章中，我们将深入分析这背后所有的技术内幕。

<h2 class="ordered">1. 普通递归版本的Fibonacci函数为什么这么慢？</h2>

最简单的Fibonacci函数可以使用下面这样的递归定义：

```scheme
(define (fib n)
  (if (or (= n 0) (= n 1))
    1
    (+ (fib (- n 1)) (fib (- n 2)))))
```

这是直接将Fibonacci数列的数学定义式直接翻译为Scheme代码。这样直译产生的代码运算效率是极其低效的。考虑调用为了计算`(fib 5)`，函数需要先计算`(fib 4)`和`(fib 3)`，`(fib 4)`的计算依赖于`(fib 3)`和`(fib 2)`，而`(fib 3)`的计算依赖于`(fib 2)`和`(fib 1)`。由于函数的调用轨迹是独立的，`(fib 4)`和`(fib 3)`计算`(fib 2)`的结果并不能互相共享，所以`(fib 2)`调用了两次。当我们这个`n`比较大时，会产生更多的重复调用，而且计算这些重复调用会非常耗时间，这就拖慢了整个计算过程。

更严重的是，由于这种“树形计算”递归需要扩展到**边界条件**去得到基本结果，当`n`非常大时，就存在一些非常长的从树的根到叶子的路径，这实际上就是函数的调用栈——这个堆栈深度可能超过了系统所允许的范围。

<h2 class="ordered">2. 如何用记忆化加速Fibonacci函数？</h2>

我们研究了函数`fib`计算效率低下的原因：有大量无谓的重复计算。解决方法也非常简单：**把计算结果缓存起来，下次用相同参数调用函数时，我们直接返回结果即可**。需要说明的是，对函数做记忆而不影响其正确性有一个很重要的前提条件：

> **函数`f`能够被记忆的条件**  
> 函数`f`能够做记忆，当且仅当它是引用透明的纯函数。即程序的生命周期内，无论何时调用`(f x)`，其中`x`可能是单个参数，也可能是多个参数，得到的结果总是一致的。

实际上，用任何一种查找结构，存储**(参数, 结果)**这种点对即可，MIT-Scheme内建支持线性表、向量（等价于其它语言中的数组）、关联表（以**键-值**形式存储的线性表）以及哈希表。为了效率和方便起见，我们选择使用哈希表来实现记忆。

```scheme
(define table (make-hash-table))

(define (fib n)
  (hash-table/loopup table n
    (lambda (x) x)
    (lambda ()
      (if (or (= n 0) (= n 1))
        (begin
          (hash-table/put! table n 1)
          1)
        (let ((res (+ (fib (- n 1)) (fib (- n 2)))))
          (hash-table/put! table n res)
          res)))))
```

我们首先定义了一个全局的哈希表`table`。而在函数`fib`内部，我们先对哈希表做一次查询操作，函数`hash-table/lookup`的语义是这样的：

> **`hash-table/lookup hash-table key if-found if-not-found`**  
> `if-found`必须是一个单参函数，`if-not-found`必须是一个无参函数。如果`hash-table`中存在以`key`为键的关联，那么`key`对应的值将会作为参数调用`if-found`，否则直接调用无参函数`if-not-found`。但无论哪个过程被调用，被调用过程的值将作为函数`hash-table/lookup`的值返回。

在本例中，如果我们在哈希表中存在键`n`，那么我们直接用一个恒等函数`(lambda (x) x)`返回该值即可（注意，键`n`对应的值会作为参数调用这个恒等函数，所以这么做有效）。如果键不存在，我们的工作会稍微麻烦一点：如果参数`n`是边界条件，那么在返回基准情况的同时，还要在哈希表中记录参数与求值结果的关联；如果参数不是边界条件，那么我们先要递归调用`(fib (- n 1))`和`(fib (- n 2))`来求得子问题的解，并将结果求和，得到本问题的解，最后再将此次求解的**(参数, 结果)**关联保存在哈希表中即可。

<h2 class="ordered">3. 如何利用词法闭包实现@memoize装饰器？</h2>

我们利用了一个全局哈希表快速实现了一个记忆化的`fib`函数，但实际上，这之中还存在一些值得商榷的问题。首先，我们使用了一个全局变量，引入的这个新名字污染了顶层环境——这是不好的编程实践。其次，对函数做记忆是一种高阶的、通用的基本模式，它的行为并不依赖于被记忆函数干了什么。

第一点告诉我们要使用某种技术限定用于记忆的哈希表`table`的作用域。第二点告诉我们要定义某种通用的操作，某种工厂。如果我们以函数为原料送入工厂，不知咋滴，我们就会得到一个被“包装”的新函数，而且神器的具有记忆之前计算结果的功能——这也就是我们所谓的“装饰器”。

> **装饰器**  
> 装饰器并不是什么神秘的概念，实际上装饰器就是一个高阶函数。它的一种普遍行为是：接收一个函数，返回一个新函数。也就是装饰器会在旧函数周围添加一些自定义代码，并返回这个新生成的函数。在上述的行为中，装饰器并不会改变原函数本身，甚至原函数接收参数的个数，但如果需要，你可以自由定制装饰器的行为。

要在Scheme中实现`@memoize`装饰器非常简单，它是一个函数定义：

```scheme
(define (@memoize func)
  (let ((table (make-equal-hash-table)))
    (lambda arg
      (hash-table/lookup table arg 
        (lambda (x) x)
        (lambda ()
          (let ((res (apply func arg)))
            (hash-table/put! table arg res)
            res))))))
```

注意，在`@memoize`函数的定义中，我们先用`let`创建了一个新的环境，并在这个环境中定义了`tabel`。这样，每个被记忆的函数都有一个属于自己的`table`。`@memoize`返回了一个`lambda`生成的新闭包，也就是被装饰后的函数。

为了讨论的方便，我们定义`(define memo-fib (@memoize fib))`。如果我们调用`(memo-fib 30)`，会发现效果并不是十分明显。这是因为被记忆后的`memo-fib`函数会调用未被记忆的`fib`函数来计算结果，而并非按照我们期望的那样去递归地调用记忆后的`memo-fib`函数。

解决方法很简单：重新绑定`fib`，可以用下面的代码实现：

```scheme
(define fib (@memoize fib))
```

同样的，我们可以定义一个新语法`@memoize!`来实现破坏性的函数记忆化：也就是说，这个调用会破坏函数原有的绑定。

```scheme
(define-syntax @memoize!
  (syntax-rules ()
    ((_ func)
     (set! func (@memoize func)))))
```

<h2 class="ordered">4. 词法闭包的一个小细节</h2>

我们似乎完完成了装饰器的编写，而且它运行良好。但有几朵阴云仍旧在这座大厦边飘荡，还有些问题需要我们解决。

1. `(let ((res (apply func arg)))`中的`func`究竟是原函数还是被记忆后的函数？还是会因`@memoize`调用而改变？
2. 执行`(@memoize! fib)`后，`fib`函数内部的所有`fib`函数究竟是原来的`fib`还是被记忆后的`fib`？
3. 整个函数调用轨迹是怎么样的？

在某个环境中，我们先定义好`fib`过程，然后再对它求值：

```scheme
1 ]=> fib

;Value 13: #[compound-procedure 13 fib]
```
我们发现，与标识符`fib`绑定的时第13号复合过程，该过程也与名字`fib`绑定了。我们将`fib`记忆化后得到的新函数与名字`memo-fib`绑定，注意到14号过程`memo-fib`确实是一个新的过程，但与它绑定的过程是匿名的。

```scheme
1 ]=> (define memo-fib (@memoize fib))

;Value: memo-fib

1 ]=> memo-fib

;Value 14: #[compound-procedure 14]
```

尝试重新改换`fib`的绑定，我们可以通过之前实现的`@memoize!`语法来实现：

```scheme
1 ]=> (@memoize! fib)

;Value 13: #[compound-procedure 13 fib]

1 ]=> fib

;Value 15: #[compound-procedure 15]
```

现在，`fib`被绑定到了15号过程上。需要说明的是，14号过程和15号过程的代码是相同的，只是14号过程是与一个新名字绑定起来了，而15号过程绑定的是原有的`fib`名字，这也就为后面的魔法埋下了伏笔。为了后面表述方便，用`fib:13`表示原始`fib`过程，用`fib:15`表示被记忆后、重新绑定到`fib`这个符号上的过程。

那么，`fib:15`过程的**体**是什么样的呢？过程`fib:15`的体是下面这样的一个`lambda`表达式：

```scheme
(lambda arg
  (hash-table/lookup table arg 
    (lambda (x) x)
    (lambda ()
      (let ((res (apply func arg)))
        (hash-table/put! table arg res)
        res))))  
```

这个表达式中的`func`存储于求值`(@memoize fib)`时所扩展的环境，因此这里的`func`始终指代调用`(@memoize fib)`时所传递的`fib:13`。如果这里的`func`指代的是记忆后的`fib:15`，那么我们的调用就会陷入没有结果的无穷循环（`fib:15`只负责查表，并不执行实际计算，所以无穷递归调用`fib:15`得不到我们想要的结果）。

那么原始`fib:13`过程中又有什么奇异的地方呢？在我们改换`fib`的绑定之前，没有什么奇怪的问题，但改换`fib`的绑定以后我们会产生这样的疑问，`fib:13`过程的体中，`fib`是指`fib:13`还是`fib:15`？

```scheme
(if (or (= n 0) (= n 1))
  1
  (+ (fib:?? (- n 1)) (fib:?? (- n 2)))))
```

在本例中，在这个很特殊的情况下，`fib:13`调用的是`fib:15`。这是因为在定义`fib`时，我们并不知道`fib`这个符号所绑定的是什么东西，需要在运行时求值确定，但是我们后来改换过`fib`的绑定，因此，这里的`fib:??`在求值时被确定为了`fib:15`。如果有必要，我们还可以再次改换`fib`的绑定，`fib:13`中的`fib:??`始终会在运行时重新求值被绑定到了哪里。

现在，是时候理清楚我们的函数调用了。这里，我们以`(fib 5)`为例，看看这里面到底发生了什么。由于Scheme标准里面并没有严格限定参数的求值顺序（可能从左到右求值，也可能反之），这里我们假设它是从左到右求值的。

1. `(fib:15 5)`先查`table`，当然，此时`table`为空，所以调用`(fib:13 5)`；
2. `(fib:13 5)`会调用`(fib:15 4)`和`(fib:15 3)`来计算子问题；
3. `(fib:15 4)`查表无果，调用`(fib:13 4)`；
4. `(fib:13 4)`调用`(fib:15 3)`和`(fib:15 2)`；
5. `(fib:15 3)`查表无果，调用`(fib:13 3)`；
6. `(fib:13 3)`调用`(fib:15 2)`和`(fib:15 1)`；
7. `(fib:15 2)`查表无果，调用`(fib:13 2)`；
8. `(fib:13 2)`调用`(fib:13 1)`和`(fib:13 0)`，由于这两个调用是边界条件，所以返回结果2，注意两个边界条件的运算结果也被记录在了表中；
9. 从8返回到7，记录调用结果`(2 . 2)`，这意味着：`(fib 2)`的结果为2；
10. 从7返回到6，`(fib:15 1)`查表时直接查到在8时记录的边界条件，因此不用递归调用`fib:13`，直接返回1，所以`(fib:13 3)`返回2+1=3，并记录结果；
11. ...

这个函数调用过程给我们这样一个概念：**两个叫做`fib`的函数，一个负责查表，一个负责实际运算**。我们用了词法作用域的小trick，让他们自如地切换身份。这是一个非常神奇而有趣的trick。

<h2 class="ordered">5. 基准测试</h2>

`fib`的重复调用并不是那么多，为了得到明显的对比效果，我们测试`ack`函数（该函数的定义在Quiz. #1中已经给出）。下面定义了一种Chez Scheme风格的`time`语法，用于执行基准测试，其行为也是依赖于MIT-Scheme内建的`with-timings`，只是做了简单的语法封装。

```scheme
(define-syntax time
  (syntax-rules ()
    ((_ thunk)
     (with-timings
       (lambda () thunk)
       (lambda (run-time gc-time real-time)
         (let ((gc-time   (* (internal-time/ticks->seconds gc-time) 1000.0))
               (cpu-time  (* (internal-time/ticks->seconds run-time) 1000.0))
               (real-time (* (internal-time/ticks->seconds real-time) 1000.0)))
           (format #t "~@8A ms elapsed cpu time~%"  cpu-time)
           (format #t "~@8A ms elapsed real time~%" real-time)
           (format #t "~@8A ms elapsed gc time~%"   gc-time)))))))
```

我们用这个函数来分别测试带记忆和不带记忆的`(ack 3 9)`函数，得到了下面有趣的结果：

```scheme
1 ]=> (time (ack 3 9))
  11770. ms elapsed cpu time
  11882. ms elapsed real time
     20. ms elapsed gc time
;Value: 4093

1 ]=> (@memoize! ack)

;Value 13: #[compound-procedure 13 ack]

1 ]=> (time (ack 3 9))
     40. ms elapsed cpu time
     38. ms elapsed real time
      0. ms elapsed gc time
```

是什么造就了如此大的差异呢？为了弄清楚这之中的原委，我们定义一个`@count-call!`的装饰器，用于为函数添加调用计数功能。如果`f`被调用计数`@count-call!`装饰后，调用`(f 'reset!)`会清空这个计数，调用`(f 'counts)`会返回这个计数，其它情况则被认为是标准的函数调用（这是《计算机程序的构造和解释》上一道习题的变形，参见**习题3.2**）。

```scheme
(define (@count-call func)
  (let ((count 0))
    (lambda arg
      (cond ((eq? (car arg) 'reset!) (set! count 0))
            ((eq? (car arg) 'counts) count)
            (else
              (set! count (+ count 1))
              (apply func arg))))))
              
(define-syntax @count-call!
  (syntax-rules ()
    ((_ func)
     (set! func (@count-call func)))))
```

我们在交互式环境中做测试，查看一下`(ack 3 9)`在两种不同情况下会产生多少次调用。

```scheme
1 ]=> (@count-call! ack)

;Value 15: #[compound-procedure 15 ack]

1 ]=> (ack 3 9)

;Value: 4093

1 ]=> (ack 'counts)

;Value: 11164370

;--------------------------------------------
; Restart MIT-Scheme to initialize the env
;--------------------------------------------

1 ]=> (@count-call! ack)

;Value 14: #[compound-procedure 14]

1 ]=> (ack 3 9)

;Value: 4093

1 ]=> (ack 'counts)

;Value: 12294
```

结果是很明显的记忆后的`ack`的调用次数重11,164,370次减少到了12,294次，后者大概是前者的0.11%。考虑到哈希表查找的时间复杂度为O(1)，记忆后的效果提升非常明显。

<h2 class="ordered">参考资料</h2>

[2]里面仔细讨论并比较了Fibonacci数列的几种求解方法。

1. [Abelson H, Sussman G J. Structure and interpretation of computer programs[J]. 1983.](http://mitpress.mit.edu/sicp/)  
2. [计算斐波纳契数，分析算法复杂度 · GoCalf Blog](http://www.gocalf.com/blog/calc-fibonacci.html)