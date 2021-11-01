---
layout: post
title: "Copilot：宇宙第一 Stack Overflow 初体验"
subtitle: "GitHub Copilot is so cool and crazy"
modified: 2021-10-28 21:28:55 +0800
tags: [github, copilot, ai auto completion]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
disqus: y
share: y
---

最近取得了 GitHub Copilot 的试用资格，于是赶紧尝新了一把。

<img src="/images/post/copilot42.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

## 使用 Copilot 之前……

刚好手上在实现一个小玩具，主要功能之一是基于符号化的方法求解表达式的导函数。我已经将部分函数实现为了 `UnaryFunc` 的实例，而这个类要求程序员在初始化时提供四个参数：

1. `symbol`：以字符串表示的函数符号；
2. `evaluate`：给定自变量的值后要如何计算函数值，应该返回一个数值；
3. `derivexpr`：如何构造函数的导函数，应该返回一个 `Expr` 类型的表达式；
4. `latexit`：如何将函数格式化为 LaTeX 代码。

<img src="/images/post/copilot0.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

在引入 Copilot 之前，我就在程序库中实现了 `MathLn`、`MathSin` 以及 `MathCos` 。如你所见，它们都十分简单。可是，当激活 Copilot 之后，一切都变得疯狂了……

## 为什么 Copilot 很酷？

在激活 Copilot 之后，当我准备实现 `MathSqrt` 的时，神奇的事儿发生了：我只键入了 `MathSqrt` ，Copliot 就帮我补齐了 531 - 535 行的代码。我不记得 `MathSqrt` 的 `derivexpr` 属性是如何补全的，但应该也是一个错误答案，由于这里并不急着用这个属性，我就放了一个代码用于占位，并加上了一个 `FIXME` 的注释，当然，后面的内容也是 Copilot 帮我补全的，真是有点冷幽默。

<img src="/images/post/copilot1.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

说实话，到这一步我并不是很吃惊，因为我前面实现了三个函数，已经具有很明显的模式了。我认为 AI 能够很容易地捕获这些重复的模式。

<img src="/images/post/copilot1_mathpow.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

真正让我吃惊的是 537 行：我键入 `MathPow` 后，它提示我这应该是 `BinaryFunc` 的一个实例，然后它的 `symbol` 应该为 `"pow"` ！需要指出的是，`BinaryFunc` 是我之前就定义好的一个类，而不是 Copilot 引入或要求我定义的类，也就是说：

1. 它知道 `MathPow` 语义上就应该与之前的 `UnaryFunc` 不一样，它需要两个参数；
2. 它应该用我们定义的 `BinaryFunc` 来表示。

紧接着，Copilot 指导我完成 `MathPow` 的定义，它快速地给出我如下建议：

<img src="/images/post/copilot2.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

不知道读者是否能够体会我当时的心境，但却确实大受震撼。不是因为它正确地定义了 `evaluate` ，我也没有受的瑕疵的 `latexit` 影响，我盯着 `derivexpr` 的定义看了半晌：

1. **Copilot 理解了它是需要对表达式求导**，并且给出了一个“几乎正确”的解！之所以是“几乎正确”，因为 Copilot 看起来确实给出了指数函数的求导。但对于本例来说，复合函数求导的“链式法则”是放在其他地方处理的，`BinaryFunc` 只需要关心本级结构的导函数构造。因此 Copilot 给出的结果中 `y: mul(y, mul(x` 处，多乘以了一个 `x`，并且就算真要加上 `x` 作为因子，也应该是 `x` 的导函数；
2. **Copilot 习得了我们程序库中用于构建表达式的方法！**我们的程序库中，预先定义了几个构造函数，如图中所示的 `mul`、`sub` 等。Copilot 知道应该使用这些函数构造出 `Expr` 类型的表达式！由于我在实现这些构造函数时，提供了类型标注，所以我不确定这是完全机器学习的结果，还是结合了类型系统而完成的程序综合。这个结果是相当震撼的。

尽管我们的程序是一段 Python 程序，由于我们向里面加入了各种基本原语，我们不妨认为我们创造了一个新的语言 `P1`。因此，Copilot ① 能够“理解”以我们的意图，② 能够将正确解法（也许是来自于 Python 语言，也有可能是其它语言）“翻译”为 `P1` 语言中合法的句子。**从这个角度来看 Copilot，不是把它看做一个针对特定语言的程序综合任务，而是看做一个就像“英译汉”那样的翻译任务，这个视角是挺新奇的。**

<img src="/images/post/copilot3.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

更好玩的是，我只要求补全 `MathPow` ，Copilot 却顺带帮我完成了 `MathExp` 和 `MathLog` 的定义！读者可以检验一下 Copilot 给出的解法是否正确。

## Copilot 似乎还可以更疯狂一点

如果读者认为上面的例子比较 trivial ，毕竟产生的都是只有三、四行的代码片段。这里还有一个例子，可以产生似乎不那么 trivial 的代码。

我想要定义一个类 `FuncCallExpr` ，实现类似于 Lisp 中 `apply` 函数的效果，这样，我们可以尽量统一 `UnaryFunc` 和 `BinaryFunc` 的代码。由于我们给定 `FuncCallExpr` 继承自 `Expr` ，那么就应该实现父类所要求的代码，下面是 Copilot 给出的答卷，跟我预期的结果真是差得八九不离十了！

<img src="/images/post/copilot4.png" alt="" class="has-shadow img-margin display" style="width: 85%;">

## 沉思 Copilot 的意义，以及程序设计的未来

或许 Copilot 告诉我们，程序员只是无情的复读机器而已？Copilot 的成功，暗示着程序设计领域中似乎充满着大量的重复模式，程序员只是有意或无意地用 A 语言或 B 语言生产这些重复模式罢了。对于具有创造力的程序员来说，如果自己想要实现的代码被 Copilot 成功补全，就会像新衣服撞衫一样令人难堪。编写那些不能够被自动补全的代码，发现新的非重复模式，似乎将成为程序设计的重要目标。

**集成 Copilot 之后，Visual Studio Code 也许就是最好的 Stack Overflow 了**。GitHub、Stack Overflow，程序员的两大生产力神器，现已全被纳入微软麾下。在与“宇宙第一编辑器” Visual Studio Code 结合之后，Copy & Paste 的两次敲击（也许还有几次鼠标点击），被一次 Tab 键给无缝、无痛代替，编码体验从未如此丝滑。

通过先给出规范（Spec），在进行编程以满足规范的**行为驱动开发（Behavior Driven Development）**或**测试驱动开发（Test Driven Development）**的开发方式将变得更加主流与有效。描述规范是一件很难的事儿，在程序综合中，通常有一套领域专用语言（比如类型系统），用来描述程序期望的行为，这些语言通常具有一定的学习成本。现在，Copilot 允许程序员用自然语言描述或从函数命名中推断期望的行为，这将极大程度上减少程序员的心智负担，并提高编码的生产力。

在计算机科学中，自指是所有怪异问题的根源，因此我们不难发问：

1. 通过不断的 hint ，能否利用 Copilot 补全出一个 Copilot 呢？
2. 进一步的，是否能够将 Copilot 运用于其自身，以进化出 The Ultimate Copilot？

作为一个坚定的符号主义者，很难让我承认 Copilot 就是最终的答案，但它确实为程序设计打开了一扇充满想象的大门……