---
layout: post
title: "Learning-SICP-趣题 #3. 美观输出简单代数表达式"
subtitle: "Learning-SICP Quiz #3. Simple Algebra Expression Pretty-Print"
formatted_title: "Learning-SICP 趣题 #3.<br />美观输出简单代数表达式"
modified: 2015-03-13 00:31:27 +0800
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

+ Quiz难度：★★★★☆
+ 涉及章节：《计算机程序的构造和解释》2.2.4 实例：一个图形语言
+ 涉及知识点：依赖反转 符号计算 分情况分析

## 绪言

在很多时候，我们希望美观地排印数学表达式，让他们优雅地显示在屏幕上。处于这个目的，著名计算机科学家、图灵奖获得者 Donald Ervin Knuth 教授发明了 $ \TeX $ 排版系统，另一位计算机科学家 Leslie Lamport 教授则在其基础上开发了宏包 $ \LaTeX $，使得我们能够方面地排印数学公式、印刷制品。

TeX/LaTeX 代码是形如下面这样的控制命令：

```ruby
\frac{ -b \pm \sqrt{b^2-4ac} }{2a}
```

它能够处理为下面这样的公式：

<div>
$$ \frac{-b\pm\sqrt{b^2-4ac}}{2a} $$
</div>

## 挑战

实际上，为了得到美观的输出，我们并不需要使用这么复杂的控制语句。请考虑下面这样的一个S-表达式：

```scheme
(/
  (+ (/ a
        (* b c))
   	 (/ 1 n))
   3))
```

它对应的代数表达式是这样的：

<div>
$$ \frac{\frac{a}{b * c} + \frac{1}{n}}{3} $$
</div>

为了方便起见，我们称像**式(2)**那样，只含有数字、字母和四则运算的代数表达式称为**“简单代数表达式”**，本节的末尾给出了该语言的上下文无关文法描述（可以暂时忽略$ constant $ 的定义）。在某些特殊情况下，我们的输出设备并不能让我们使用像素级的绘制，我们也很难在这种设备上绘制出像式(2)那样的效果。对于提供字符级绘制的设备，我们可以绘制出像下面这样的式子：

```ruby
    a       1
 ------- + ---
  b * c     n
---------------
       3
```

我们的任务就是设计一个以字符形式美观输出简单代数表达式的系统，这个系统接受合法的S-表达式作为输入，并将其“美观的绘制”在屏幕上。当然，还有一些实现上的细节你需要注意，请参考下一节**注意事项**。下面是**“简单代数表达式”**的上下文无关文法。

<div>
$$
\begin{align*}
  \left< number \right> \longrightarrow  & \; \text{ any Scheme number } \\
  \left< variable \right> \longrightarrow  & \; \text{ any Scheme symbol } \\
  \left< constant \right> \longrightarrow  & \; \pi \: |\: e\: |\: \hbar  \\
  \left< atom \right> \longrightarrow  & \; \left< variable \right> |\left< constant \right> |\left< number \right>  \\
  \left< operator \right> \longrightarrow  &  \; + \;| \; - \; | \; \ast \;| \; \; /  \\
  \left< exp \right> \longrightarrow & \; \left< atom \right> |
   \; ( \left< operator \right> \left< exp \right> \left< exp \right>  )
\end{align*}$$
</div>


## 注意事项

S-表达式的优先级是确定好的，在还原成简单代数表达式时，请注意正确还原优先级。例如，S-表达式`(* (+ 1 2) (- 3 4))`所对应的正确简单代数表达式是$ (1 + 2) * (3 - 4) $，而不是$1 + 2 * 3 - 4$。

输出的简单代数表达式应符合习见约定，如S-表达式`(+ (+ 1 2) (- 3 4))`应为$1 + 2 + 3 - 4$，而非$(1 + 2) + (3 - 4)$或$1 + 2 + (3 - 4)$。又比如S-表达式`(/ (+ 1 2) (* 3 4))`应为$ \frac{1 + 2}{3 * 4} $，而非$ \frac{(1 + 2)}{(3 * 4)} $。

分数线应作为基线与符号对齐，但这条规则只针对同级运算数。比如下例：

```ruby
                                         1                                     
1 + ---------------------------------------------------------------------------
                         t                                                     
                        ---                                                    
                         t                                4                m   
     a * --------------------------------- * (------------------------- + ---) 
                              1                             9              n   
                             ---               5 + -------------------         
                   x          2       1                      16                
          1 + ----------- + ----- - -----           7 + -------------          
               a + b + c      3       2                  (x + y) * a           
                                     ---                                       
                                      3                                        
```

## 测试用例

下面给出几个可以用于测试的用例。

```scheme
(define pi '(+ (/ (/ (/ a b) e) b)
               (/ 1
                  (+ 3
                     (/ 4
                        (+ 5
                           (/ 9
                              (+ 7
                                 (/ 16
                                    (* (+ 9 a) b))))))))))

(define t1 '(+ 1
               (/ 1
                  (* (* a
                        (/ (/ t t)
                           (+ (+ 1
                                 (/ x
                                    (- a
                                       (- b c))))
                              (- (/ (/ 1 2)
                                    3)
                                 (/ 1 (/ 2 3))))))
                     (+ (/ 4
                           (+ 5 (/ 9
                                   (+ 7
                                     (/ 168
                                        (* (+ x y) a))))))
                        (/ m n))))))

(define t2 '(* 1 (- (/  c (+ a b)) 2)))

(define t3 '(* 1 (- (+  c (+ a b)) 2)))
```

