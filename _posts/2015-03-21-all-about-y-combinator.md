---
layout: post
title: "魂断不动点——Y组合子的前世今生"
subtitle: "All About Y Combinator"
modified: 2015-03-21 19:57:59 +0800
tags: [Y Combinator, Lisp, Scheme, Functional Programming, lambda, Y组合子, 不动点, lambda演算, 函数式编程]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
mathjax: y
disqus: y
---

## 序言

关于$\mathcal{Y}$组合子和$\lambda$-演算相关的介绍资料实在太多了，但它们似乎都不太完全。既然$\mathcal{Y}$组合子的全称为*$\mathcal{Y}$不动点组合子*，那么它至少包含了*不动点*和*组合子*两种属性。本文作为一个综合性的论述，试图回答关于$\mathcal{Y}$组合子的几个问题，包括：

1. 组合子是什么？
2. 引入组合子的动机是什么？
3. 不动点是什么？
4. $\mathcal{Y}$组合子是如何推导出来的？
5. 不动点组合子是唯一的么？
6. 如何在 Call-By-Value 的语义中实现$\mathcal{Y}$组合子？

需要说明的是，为了让我们的讨论更加严谨，我们非常小心地用形式化方法自习地定义了朴素$\lambda$-演算，造成的结果是整片文章充斥了大量的数学定义与推导。读者不必深究于其中的数学定义，只需要关注核心思想即可。

## 初识不动点

定义函数 $f \colon \mathbb{R} \mapsto \mathbb{R}$，如果 $\exists x \in D(f)$，使得 $ x = f(x) $，那么我们称点 $x$ 是函数 $f(x)$ 的不动点。

容易发现，根据等式 $ x = f(x) $，我们可以进行下面这样的无穷代换：

<div>
$$
\begin{align}
	x & = f(x) \\
	\label{def:fixed1-begin}
	  & = f(f(x)) \\
	  & = f(f(f(x))) \\
	  & = \dots
	\label{def:fixed1-end}
\end{align}
$$
</div>

初等函数的不动点非常好求。我们只需要将将等式 $ x = f(x) $ 稍作移项处理，就可以得到方程 $ f(x) - x = 0 $，这里我们就把寻找不动点的问题归约为了方程求根问题。


## $\lambda$-演算简介

$\lambda$-演算旨在考察函数的计算性质，它是将函数看作变量的更名代换规则而不是看作图像的一种不带类型的理论模型。它最初由 Alonzo Church 和他的学生 Stephen Cole Kleene 在20世纪30年代引入。最开始，Church 试图创造一套完整的形式系统作为数学的基础，当他随后发现这个系统易受罗素悖论的影响。虽然如此，$\lambda$-演算中涉及整数的那部分是完全成功的。利用这个理论 Church 将“可计算问题”形式化地定义为“$\lambda$-可定义”。Kleene 在随后的研究中，证明了$\lambda$-可定义等价于 Godel - Herbran 的递归性。

需要指出的是，$\lambda$-演算的内核十分之小，*朴素$\lambda$-演算(Naive $\lambda$-Calculus)*或称*无类型$\lambda$-演算(Type-Free $\lambda$-Calculus)*可以用下面的文法来描述：

<div>
$$
\begin{align}
\left< identifier \right>        & \longrightarrow \; a \; | \; b \; | \; \dots \; | \; z \label{lambda:atom} \\
\left< abstraction \right> & \longrightarrow \; \lambda \left< identifier \right> . \left< \lambda \! - \! exp \right> \label{lambda:abstraction} \\
\left< application \right> & \longrightarrow \; (\left< \lambda \! - \!exp \right>)\left< identifier \right> \label{lambda:application}\\
\left< \lambda \! - \! exp) \right> & \longrightarrow \; \left< identifier \right> \; | \; \left< abstraction \right> \; | \; \left< application \right> \\
\end{align}
$$
</div>

在有的文献中式[$\ref{lambda:atom}$]也被称作*原子(Atom)*，因为它们都是最小而不可再细分的元素，我们可以很容易理解它们的“意思”。式[$\ref{lambda:abstraction}$]被称为*抽象规则(Abstraction Rule)*，它是$\lambda$-演算中**构造函数的唯一方式**。式[$\ref{lambda:application}$]被称为*应用规则(Application Rule)*，它是抽象规则的逆过程，也是复合函数的实质计算过程。然而，应用规则的语义并不像我们看上去的那么“显然”，我们将在后面仔细考察这个问题。

这里需要注意一下，式[$\ref{lambda:application}$]的记法不同于习见的 S-表达式或 M-表达式的记法，我们采用的记法是在执行“应用”操作的函数两边加括号，这样不但使“应用”操作变成*右结合的(Right Associative)*，同时也可以避免大部分情况下括号的密集出现：

<div>
$$
\begin{align*}
\text{ S-Expression} & \colon \Big{(}\big{(}\lambda a. \lambda b. \lambda c.(a \; (b \; c))\big{)} \; \big{(}(\lambda x. \lambda y. x) \; (\lambda z.z \; v)\big{)}\Big{)}\\
\text{ M-Expression} & \colon \lambda a. \lambda b. \lambda c.a[b[c]]\Big{[}\lambda x.\lambda y.x\big{[}\lambda z.z[v]\big{]}\Big{]} \\
\text{ Our Notion}   & \colon (\lambda a. \lambda b. \lambda c.(a)(b)c)(\lambda x.\lambda y.x)(\lambda z.z)v  \\ 
\end{align*}
$$
</div>

按照我们的记法，式[$\ref{def:fixed1-begin} \sim \ref{def:fixed1-end}$]可以改写为：

<div>
$$
\begin{align}
	x & = (f)x \\
	\label{def:fixed2-begin}
	  & = (f)(f)x \\
	  & = (f)(f)(f)x \\
	  & = \dots
	\label{def:fixed2-end}
\end{align}
$$
</div>

下面将考察$\lambda$-演算的语义。首先我们需要定义*全等（Identical）*这个概念，我们可以按照下面的方式归纳地定义全等，其中 $P$ 和 $Q$ 都是任意的$\lambda$-表达式：

<div>
$$
\begin{align}
   (i)\; & \varphi \equiv \psi, \; \text{如果} \varphi \text{和} \psi \text{是同一个字母} \\
  (ii)\; & \lambda \varphi .P \equiv \lambda \psi .Q , \; \text{如果} \varphi \equiv \psi \text{且} P \equiv Q \\
 (iii)\; & (P)\varphi \equiv (Q)\psi , \; \text{如果$\varphi \equiv \psi $且$P \equiv Q$}
\end{align}
$$
</div>

显然，两个全等的表达式，其语义是完全相同的。下面考虑*易名规则(Renaming Rule)*，记号是 $\\{ \varphi/\psi \\}P$ ，表示将 $P$ 中所有出现 $\psi$ 的地方都替换为 $\varphi$ ，其中 $\varphi \in \left< identifier \right>$ 且 $\psi \in \left< identifier \right>$ 。其形式化定义如下，其中，$P,Q$都是任意的$\lambda$-表达式：

<div>
$$
\begin{align*}
		(i) & \; \{ \varphi / \psi \} \psi \equiv \varphi \\ 
	   (ii) & \; \{ \varphi / \psi \} \omega \equiv \omega , \text{如果$\psi \not \equiv \omega$} \\
	  (iii) & \; \{ \varphi / \psi \} \lambda \psi . P \equiv \lambda \varphi. \{ \varphi / \psi \} P \\
	   (iv) & \; \{ \varphi / \omega \} \lambda \psi . P \equiv \lambda \psi. \{ \varphi / \omega \} P , \text{且$\omega \not \equiv \psi$} \\
	    (v) & \; \{ \varphi / \psi \} (P)Q \equiv (\{ \varphi / \psi \}P)\{ \varphi / \psi \} Q\\ 
\end{align*}
$$
</div>

为了考察应用规则的语义，我们还需要先考察自由变量的定义，记号 $\phi(E)$ 表示$\lambda$-表达式 $E$ 中所有自由变量组成的集合，它可以按照下面的方式归纳定义：

<div>
$$
\begin{align}
		(i) & \; \phi (\psi) = \{ \psi \}, \psi \in \left< identifer \right> \\
	   (ii) & \; \phi (\lambda \psi.P) = \phi (P) - \{ \psi \} \\
	  (iii) & \; \phi ((P)Q) = \phi (P) \cup \phi (Q) \\  
\end{align}
$$
</div>

我们说 $\psi$ 在$\lambda$-表达式 $P$ 中自由出现，当且仅当 $\psi \in \phi(P)$ ，此时称 $\psi$ 为*自由变量(Free Variable)*，否则称为*绑定变量(Bounded Variable)*或*约束变量*。

例如，在表达式 $(\lambda {\color{blue}x}. \lambda {\color{orange}y}.({\color{blue}x}){\color{orange}y})\lambda {\color{green}x}.({\color{red}y}){\color{green}x}$ 中，变量 ${\color{blue}x}$ 和变量 ${\color{green}x}$ 都是约束变量，虽然二者都是同一个字母，但并不是同一个约束变量，因为它们被不同的$\lambda$-抽象绑定着，有着各自的辖域（已经用颜色标记出来）。而变量 ${\color{red}y}$ 是自由变量，而 ${\color{orange}y}$ 是约束变量，因为它被 $ \lambda {\color{orange}y}$ 所绑定。所以 $\phi\big{(}(\lambda x. \lambda y.(x)y)\lambda x.(y)x \big{)} = \{ { \color{red} y }\}$ 

理解$\lambda$-演算中变量绑定的辖域非常重要。在定义清楚自由变量和绑定变量以后，我们开始介绍*$\alpha$-变换($\alpha$-conversion)*。

<div class="definition">
（$\alpha$-变换）$\lambda \varphi.P \rightarrow _{\alpha} \lambda \psi.\{\psi / \varphi\}P,$ 其中 $\psi$ 是 $P$ 中没有出现过的符号。
</div>

$\alpha$-变换有个很直观的解释：*绑定变量的名字不重要*。这在习见程序设计语言中被解释为：*局部变量只是占位符，外部对其名字不敏感*。但是需要注意$\alpha$-变换同时也强调不能将绑定变量更名为子表达式中出现过的符号，否则就会将子表达式中的自由变量意外地变成约束变量，或者提升了约束变量的约束辖域。

为了定义$\beta$-归约，我们需要先要引入*代换规则(Substitution Rule)*。请读者注意代换规则与易名规则一个显著的不同：*易名规则只允许用原子符号去替换原子符号，而代换规则允许用复合$\lambda$-表达式去替换原子符号*。为了同易名规则区别开，我们使用记号 $[P/\varphi]Q$ 表示用$\lambda$-表达式 $P$ 代换 $Q$ 中所有自由出现的 $\varphi$ ，其中 $ \varphi \in \phi (Q) $ ，$P,Q$ 是任意的$\lambda$-表达式。

<div class="definition">
（代换规则） 代换规则是按下面的规则归纳定义的，其中$P$、$P_1$、$P_2$和$Q$都是任意的$\lambda$-表达式：
$$
\begin{align}
		(i) \; & [Q / \varphi]\varphi \cong Q \\
	   (ii) \; & [Q / \varphi]\psi    \cong \psi, \text{如果$\varphi \not \equiv \psi$} \\
	  (iii) \; & [Q / \varphi]\lambda \varphi.P \cong \lambda \varphi.P \\
	   (iv) \; & [Q / \varphi]\lambda \psi. P \cong \lambda \psi.[Q / \varphi]P, \text{如果$\varphi \not \equiv \psi$,} \label{sub1}
	           \text{且$\varphi \not \in \phi(P) \vee \psi \not \in \phi(Q)$为真} \\
	    (v) \; & [Q / \varphi]\lambda \psi. P \cong \lambda \omega.[Q / \varphi]\{\omega / \psi \}P, \text{其中$\omega \not \equiv \varphi \not \equiv \psi$,} \notag \\
	    & \text{且$\omega$没有在$(P)Q$中出现过,如果$\varphi \not \equiv \psi$且$\varphi \in \phi(P) \wedge \psi \in \phi(Q)$为真} \label{sub2}\\
	   (vi) \; & [Q / \varphi](P_1)P_2 \cong ([Q / \varphi]P_1)[Q / \varphi]P_2 \\
\end{align}
$$
</div>

代换规则中的式[$\ref{sub2}$]还使用到了易名规则，下面给出一个具体的例子来说明这条规则。

<div class="example">
	根据代换规则计算代换表达式 $[\lambda y.(x)y/y]\lambda x.(y)x$ 。
	<p>$\because x \in \phi \big{(} \lambda y.(x)y \big{)} \text{且} y \in \phi \big{(} \lambda x.(y)x \big{)} $</p>
	<p>$\therefore \text{引入新变量} z , \text{显然} z \not \equiv x \not \equiv y  $</p>
	<div>$\therefore \text{代换过程如下}$
		$$
		\begin{align*}
			[\lambda y.(x)y/y]\lambda x.(y)x & \cong \lambda z.[\lambda y.(x)y/y]\{z/x\}(y)x \\
			& \cong \lambda z.[\lambda y.(x)y/y](y)z \\
			& \cong \lambda z.(\lambda y.(x)y)z \\
		\end{align*}
		$$
	</div>
</div>

注意到代换规则保证*自由变量带入$\lambda$-表达式后仍然是自由变量，且约束变量的辖域不会改变*。读者可以尝试不使用易名规则引入新变量，直接将表达式带入，再验证新表达式每个变量的辖域。

下面正式引入*$\beta$-归约($\beta$-reduction)*，其中 $P,Q$ 是任意的$\lambda$-表达式：

<div class="definition">
（$\beta$-归约）$ (\lambda \varphi.P)Q \rightarrow [Q/\varphi]P $
</div>

我们将形如 $(\lambda \varphi.P)Q$ 的$\lambda$-表达式称为*$\beta$-归约式($\beta$-redex)* ，将 $Q/\varphi]P$ 的值称为*归约结果(contractum)*。直观地看，归约结果应该比归约式要简单，比如 $((\lambda x.\lambda y.(x)y)(\lambda x.x))x \rightarrow x$。但实际上有很多$\beta$-归约式它们的归约结果并不比原表达式简单，式[$\ref{inf:begin} \sim \ref{inf:end}$]演示了一个特殊的$\beta$-归约式——它的归约结果就是它自己。

<div>
$$
\begin{align}
(\lambda x.(x)x)\lambda x.(x)x & \rightarrow [\lambda x.(x)x/x]\lambda x.(x)x \label{inf:begin}\\
& \cong (\lambda x.(x)x)\lambda x.(x)x \\
& \cong (\lambda x.(x)x)\lambda x.(x)x \label{inf:end} \\
& \dots \notag \\
\end{align}
$$
</div>

不难发现，如果一个归约式的归约结果比它本身“简单”，那么可以期望这个归约过程在某个地方能够停止下来，以至于无法再应用$\beta$-归约让它变得“更简单”。在这个信念下，我们引入*范式(Normal Form)*这个概念，其中 $\varphi \in \left< identifier \right>$ ，且$P, Q$为任意的$\lambda$-表达式。

<div class="definition">
（范式） 没有$\beta$-归约式的$\lambda$-表达式，即，表达式中没有形如 $(\lambda \varphi.P)Q$ 的子表达式。
</div>

没有无穷归约的$\lambda$-表达式一定有范式，反之不尽然。下面给出一个虽然有无穷归约的子表达式，但最终仍可归约为范式的$\lambda$-表达式$^{[\ref{note1}]}$来结束本小节的讨论。

<div>
$$
\begin{align*}
	(\lambda y.(\lambda z.w)y)(\lambda x.(x)x)\lambda x.(x)x 
    & \cong (\lambda z. w)(\lambda x.(x)x)\lambda x.(x)x \\
    & \cong w
\end{align*}
$$
</div>

$[\dagger] \text{ 需要 Church-Rosser 定理来改换$\beta$-归约的顺序。} \tag{$\dagger$} \label{note1}$


## 逻辑组合子

我们在上一节强调过，在$\lambda$-演算中，区分变量是绑定的还是自由的十分重要。$\lambda$-表达式的值可能依赖于其中的自由变量，而自由变量的值则依赖于*上下文(Context)*。非正式地，我们可以把上下文想做是一个有很多“洞(Hole)”的$\lambda$-表达式，我们可以向这些洞中填入任意的$\lambda$-表达式。式[$\ref{lambda:context}$]就是一个可以作为上下文使用的$\lambda$-表达式：

<div>
$$
	C = \label{lambda:context}((\lambda x.\lambda y.{\bf hole})E)F
$$
</div>

我们尝试将$\lambda$-表达式 $\lambda x.(y)x$ 带入这个上下文，这就意味着我们要代换掉 $C$ 中的 ${\bf hole}$。

<div>
$$
\begin{align*}
	[ \lambda x.(y)x/{\bf hole} ] C & \rightarrow \, ((\lambda x.\lambda y.\lambda x.(y)x)E)F \\
	                                  & \rightarrow \, (\lambda y. \lambda x.(y)x)F \\
	                                  & \rightarrow \, \lambda x.(F)x

\end{align*}
$$
</div>

通过简单的$\beta$-归约，我们可以很容易地观察到，表达式 $\lambda x.(y)x$ 在这样的上下文 $C$ 中的语义变成了 $\lambda x.(F)x$ ，显然不同的上下文中会有不同的 $F$ ，这也就导致了$\lambda$-表达式的语义变化。自由变量好比习见程序设计语言中的全局变量，而绑定变量则像函数的形式参数。在程序设计语言中，通常提倡尽量少用甚至不用全局变量，都是为了规避不可控的上下文对局部过程的影响。

<div class="definition">
（组合子） 没有自由变量的$\lambda$表达式，即<em>闭$\lambda$-表达式(Closed $\lambda$-Expression)</em>。
</div>

*组合子逻辑*最初由 Moses Ilyich Schönfinkel 于1924年发现。Schönfinkel 认为自由变量只是在语法上起辅助作用的概念，于是为了消除数理逻辑中对变量的需要，他引入了组合子逻辑。在大约1926-1927年间， Haskell B.Curry 开始研究代换过程，通过引入跟 Schönfinkel 相同的组合子，他将代换分解为了跟细小的步骤。在1927年晚期， Curry 发现了 Schönfinkel 的研究，并称其“早已预见了我的工作”。尽管如此， Curry 并没有放弃他对组合子逻辑的研究。他游学至德国哥廷根（Gottingen），在 Bernays 指导下完成了博士论文。在论文中，他构建了一套组合子的形式化系统，并证明了组合子 $\mathcal{B},\mathcal{C},\mathcal{K},\mathcal{W}$ 是完备的。

在组合子逻辑中，有两个特殊的组合子（也称*基本组合子*） $\mathcal{S}$ 和 $\mathcal{K}$，它们的定义如下：

<div>
$$
\begin{align*}
\mathcal{S} & \cong \lambda x. \lambda y. \lambda z.((x)z)(y)z \\
\mathcal{K} & \cong \lambda x. \lambda y. x
\end{align*}
$$
</div>

Curry 试图使我们相信，组合子逻辑是完备的，通过 $\mathcal{S}$ 和 $\mathcal{K}$ 两个标准组合子的组合构造，我们可以构造出其它的函数——这就是它们被称为“组合子”的原因。例如，$\lambda$-演算中的恒等函数 $\lambda x.x$ 可以构造为 $((\mathcal{S})\mathcal{K})\mathcal{K}$，我们可以根据$\beta$-归约很容易地验证两者的等价性：

<div>
$$
\begin{align*}
(\lambda x.x)\alpha & = ((( \mathcal{S} ) \mathcal{K} ) \mathcal{K} ) \alpha \\
                    & = ((\lambda y. \lambda z.(( \mathcal{K} )z)(y)z) \mathcal{K} ) \alpha \\
                    & = (\lambda z.(( \mathcal{K} )z)( \mathcal{K} )z) \alpha \\
                    & = (( \mathcal{K} ) \alpha )( \mathcal{K} ) \alpha \\
                    & = ((\lambda x. \lambda y.x)\alpha)(\lambda x. \lambda y.x)\alpha \\
                    & = (\lambda y.\alpha)\lambda y.\alpha \\
                    & = \alpha
\end{align*}
$$
</div>


## 不动点Y组合子

请读者考虑阶乘函数 $FACT$ 的定义：

<div>
$$
\begin{align}
(FACT) n = (((\mathcal{ZERO?})n)1)((*)n)(FACT)(\mathcal{PRED})n \label{def:fact1}
\end{align}

$$
</div>

其中 $\mathcal{ZERO?}$ 和 $\mathcal{PRED}$ 都是组合子，其具体定义请参见[附录A]。部分读者可能不太适应用这种方式编码 $FACT$ 函数，这是因为通过*库里化(Currying)*来实现多参函数调用，并将  $\text{if}$ 控制流编码为组合子的方式在传统程序设计语言中并不常见。下面给出与它们等价的 Scheme 代码帮助读者理解，读者可以不用深究 $FACT$ 的定义，只需关注不动点的推导过程。需要强调的是，传递给 $\mathcal{PRED}$ 的参数是邱奇数组合子，此处读者可以略过这些操作上的细节。

<div>
$$
\begin{align*}
(((\mathcal{ZERO?})c)x)y & = \text{(if (= c 0) x y)} \\
(\mathcal{PRED})n & = \text{(- n 1)} \\ 
\end{align*}
$$
</div>

回到式[$\ref{def:fact1}$]，其中，等式左边是$\beta$-归约式，我们要将函数给“抽象”出来，可以通过向等式右边添加 $\lambda$ 项得到。因此，我们有：

<div>
$$
FACT = \lambda n.(((\mathcal{ZERO?})n)1)((*)n)(FACT)(\mathcal{PRED})n
$$
</div>

观察等式右边，我们发现仍然存在自由变量 $FACT$，我们需要再次使用$\lambda$将其抽象出来。（有读者会认为此处的 $\mathcal{ZERO?}$ 和 $\mathcal{PRED}$ 组合子也是自由变量。这种想法是错误的。这里的 $\mathcal{ZERO?}$ 和 $\mathcal{PRED}$ 只是一种简写记号。它并不依赖于任何上下文，读者可以将[附录A]中 $\mathcal{ZERO?}$ 和 $\mathcal{PRED}$ 的定义代换到这个式子中即可。）

<div>
$$
FACT = (\lambda f.\lambda n.(((\mathcal{ZERO?})n)1)((*)n)(f)(\mathcal{PRED})n)FACT
\label{def:fact2}
$$
</div>

我们可以将式[\ref{def:fact2}]看做如下表达式：

<div>
$$
F = (\mathcal{E})F
\label{def:fact3}
$$
</div>

在式[\ref{def:fact3}]中，$F$ 是 $FACT$，$\mathcal{E}$ 是组合子 $\lambda f.\lambda n.(((\mathcal{ZERO?})n)1)((*)n)(f)(\mathcal{PRED})n$ 。考虑式[\ref{def:fact3}]中的等式，可以无穷地将这个等式替换等式右边出现的 $F$，即：

<div>
$$
\begin{align}
	F & = (\mathcal{E})F \\
	\label{def:fact4begin}
	  & = (\mathcal{E})(\mathcal{E})F \\
	  & = (\mathcal{E})(\mathcal{E})(\mathcal{E})F \\
	  & = \dots
	\label{def:fact4end}
\end{align}
$$
</div>

请将式[$\ref{def:fact4begin} \sim \ref{def:fact4end}$]与式[$\ref{def:fixed2-begin} \sim \ref{def:fixed2-end}$]比较，我们不难发现，**$F$ 也是某个函数的不动点**。这里的“某个函数”，就是指组合子 $\mathcal{E}$ 。但是此时我们并没有解出 $F$ ，并且与初等函数不同，我们还没有介绍“解高阶函数方程”的方法。幸运的是，类似于一元二次方程，我们有解高阶函数不动点的“求根公式”——传说中的$\mathcal{Y}$不动点组合子。既然$\mathcal{Y}$组合子是“通用求根公式”，那么它必然满足下面的等式：

<div>
$$
\begin{align*}
	(\mathcal{Y})\mathcal{E} & = F
\end{align*}
$$
</div>

同时也要注意到 $F = (\mathcal{E})F$ 这个事实，于是我们有：

<div>
$$
\begin{align}
	(\mathcal{Y})\mathcal{E} & = \mathcal{F} \label{y-start}\\
	                         & = (\mathcal{E})\mathcal{F} \\ 
	                         & = (\mathcal{E})(\mathcal{Y})\mathcal{E} \\
	                         & = (\mathcal{E})(\mathcal{E})(\mathcal{Y})\mathcal{E} \\
	                         & = \dots \label{y-ends}
\end{align}
$$
</div>

推导$\mathcal{Y}$组合子的表达式是一项非常有创造力的工作。我们发现式[$\ref{inf:begin} \sim \ref{inf:end}$]的模式很像我们的$\mathcal{Y}$组合子，但它总是保持自己不变，而$\mathcal{Y}$组合子会不断增加一个前缀。于是，我们可以在式[$\ref{inf:begin} \sim \ref{inf:end}$]的基础上稍作修改，从而得到$\mathcal{Y}$组合子的表达式：

<div>
$$
\begin{align}
	\mathcal{Y} & = \lambda f.(\lambda x.(f)(x)x) \lambda x.(f)(x)x
\end{align}
$$
</div>

于是，我们就能利用“万能求根公式”$\mathcal{Y}$组合子来计算函数 $\mathcal{FACT}$的定义了。注意，这个时候解出的 $\mathcal{FACT}$ 也是一个组合子，因为它的表达式中没有自由变量：

<div>
$$
\begin{align}
\mathcal{FACT} & = (\mathcal{Y}) \mathcal{E} \\
               & = (\mathcal{Y})\lambda f.\lambda n.(((\mathcal{ZERO?})n)1)((*)n)(f)(\mathcal{PRED})n \\
               & = \dots \\
\end{align}
$$
</div>

## 一个事实：有无穷多个不动点组合子

我们给出了$\mathcal{Y}$不动点组合子，实际上还有一个非常出名的不动点组合子，即$\mathcal{T}$组合子—— Turing 不动点组合子，它的定义如下：

<div>
$$
\mathcal{T} \cong (\lambda x. \lambda y.(y)((x)x)y)\lambda x. \lambda y.(y)((x)x)y
$$
</div>

我们发现，$\mathcal{Y}$组合子和$\mathcal{T}$组合子是非常地相像，以至于我们可以发现它们之间满足下面的等式：

<div>
$$
\begin{align}
	\mathcal{T} & = (\mathcal{Y})\lambda x. \lambda y.(y)(x)y \\
	& = (\lambda f. (\lambda x.(f)(x)x) \lambda x.(f)(x)x) \lambda x. \lambda y. (y)(x)y \\
	& = (\lambda x.(\lambda x. \lambda y.(y)(x)y)(x)x)\lambda x.(\lambda x. \lambda y.(y)(x)y)(x)x \\
	& = (\lambda x.\lambda y.(y)((x)x)y)\lambda x.\lambda y.(y)((x)x)y\\
\end{align}
$$
</div>

事实上，我们可以通过下面的方式构造一个由无穷个不动点组合子序列：

<div>
$$
\begin{align*}
	\mathcal{Y_1} & \cong \mathcal{Y} \\
	\mathcal{Y_2} & \cong (\mathcal{Y_1})\mathcal{G}\\
	\mathcal{Y_3} & \cong (\mathcal{Y_2})\mathcal{G}\\
	              & \dots \\
	\mathcal{Y_n} & \cong (\mathcal{Y_{n-1}})\mathcal{G}\\
\end{align*}
$$
</div>

其中，$\mathcal{Y_2}$组合子即是$\mathcal{T}$组合子，而$\mathcal{G}$组合子它的定义如下：

<div>
$$
	\mathcal{G} = \lambda x. \lambda y.(y)(x)y
$$
</div>



## Curry悖论：无法逃离的自指怪圈

<div class="proof">
Russell 悖论等价于逻辑算符 $\neg$ 的不动点，即 $(\mathcal{Y})\neg$ 。
</div>

证：如果 $a \in A$ ，那么我们就记作 $(A)a$。

那么对于集合 $\\{x | P[x]\\}$ ，我们可以表示为 $\lambda x. P[x]$。

这样，$ a \in \\{ x | P[x]\\} $ 就变成了 $(\lambda x. P[x])a = P[a]$。

取 $R = \\{ x | x \not \in x \\} = \lambda x. (\neg)(x)x$ 。

于是有 $\forall r.[(R)r \Longleftrightarrow (\neg)(r)r ]$

于是有 $(R)R \Longleftrightarrow (\neg)(R)r$

注意到 $(R)R \cong (\lambda x. (\neg)(x)x)\lambda x. (\neg)(x)x = (\mathcal{Y})\neg$

<div class="proof">
逻辑算符蕴含 $\rightarrow$ 在$\lambda$-演算中是不一致的。
</div>

证：逻辑算符蕴含满足下面的公理：

<div>
$$
(P \rightarrow (P \rightarrow Q)) \rightarrow (P \rightarrow Q) \label{axiom1} \tag{$\Gamma_1$} 
$$
</div>

同时逻辑蕴含也要满足*肯定前件(modus ponens)*规则，即：

<div>
$$
P \rightarrow Q, P \vdash Q \tag{modus ponens} \label{mp}
$$
</div>

我们将逻辑蕴含 $\rightarrow$ 编码成我们系统中的组合子 $imp$ ，因此公理[$\ref{axiom1}$]可以编码为：

<div>
$$
((imp)((imp)P)((imp)P)Q)((imp)P)Q
$$
</div>

对任意的$\lambda$-表达式 $Q$，我们命：

<div>
$$
 N = \lambda x.((imp)x)((imp)x)Q
$$
</div>

又命：

<div>
$$
P = (\mathcal{Y})N
$$
</div>

即 $P$ 是 $N$ 的一个不动点。因此，我们得到：

<div>
$$
((imp)P)((imp)P)Q = (N)P = (N)(\mathcal{Y})N = P \label{subst}
$$
</div>

在公理[$\ref{axiom1}$]中应用式[$\ref{subst}$]定义的等式，我们得到：

<div>
$$
\begin{align}
  & ((imp)((imp)P)((imp)P)Q)((imp)P)Q) \\
= & ((imp)P)((imp)P)Q\\
= & P \label{subout} \\
\end{align}
$$
</div>

式[$\ref{subout}$]作为公理的代换结果，其值也因永真。因此对任意的 $Q$ (注意 $N$ 的定义依赖于 $Q$)，我们有

<div>
$$
	P = (\mathcal{Y})N = \mathcal{true} \label{pout}
$$
</div>

对式[$\ref{pout}$]使用[$\ref{mp}$]，我们得到：

<div>
$$
	Q = \mathcal{true}
$$
</div>

与我们前面得到的 $Q$ 为任意的$\lambda$-表达式相悖。

这个悖论不难理解。考虑到 $P \rightarrow Q \Longleftrightarrow \neg P \vee  Q$ 的事实，逻辑连词 $ \rightarrow $ 实际包含了 $\neg$ 的语义。既然在上面的讨论中，我们证明了 $\neg$ 在$\lambda$-演算中会导致矛盾，在一个蕴含 $\neg$ 的系统中导出系统不一致就不足为奇了。

## 在 Call-By-Value 语义语言中实现Y组合子

## 参考文献

* **[Abel83]** Abelson H, Sussman G J. Structure and interpretation of computer programs[J]. 1983.
* **[Baren84]** Barendregt H P. The lambda calculus[M]. Amsterdam: North-Holland, 1984.
* **[Hank04]** Hankin C. An introduction to lambda calculi for computer scientists[M]. King's College Publications, 2004.
* **[Hind06]** Cardone F, Hindley J R. History of lambda-calculus and combinatory logic[J]. Handbook of the History of Logic, 2006, 5: 723-817.
* **[Klee81]** Kleene S C. The theory of recursive functions, approaching its centennial[J]. Bulletin of the american mathematical society, 1981, 5(1): 43-61.
* **[RevG09]** Révész G E. Lambda-calculus, combinators and functional programming[M]. Cambridge University Press, 2009.

## 附录A 常见组合子的定义

在组合子逻辑中，虽然 $\mathcal{S}, \mathcal{K}$ 组合子足以构建出一个完备的系统，但通常也引入组合子 $\mathcal{I}$ 来简化某些表达式的表示，故组合子逻辑有时也称为*SKI逻辑*。

<div>
$$
\begin{align*}
\mathcal{S} & \cong \lambda x. \lambda y. \lambda z.((x)z)(y)z \\
\mathcal{K} & \cong \lambda x. \lambda y. x \\
\mathcal{I} & \cong \lambda x. x \\
            & \cong ((\mathcal{S})\mathcal{K})\mathcal{K}
\end{align*}
$$
</div>

同时，Curry 还引入了两个组合子 $\mathcal{B}, \mathcal{C}$：

<div>
$$
\begin{align*}
\mathcal{B} & \cong \lambda x. \lambda y. \lambda z.(x)(y)z \\
\mathcal{C} & \cong \lambda x. \lambda y. \lambda z.((x)z)y 
\end{align*}
$$
</div>

需要提及一些历史变迁。$\mathcal{B},\mathcal{C},\mathcal{I},\mathcal{K},\mathcal{S}$ 都是基本组合子的现代版本，在 Schonfinkel 最初的版本中，它们分别被称作 $\mathcal{Z},\mathcal{T},\mathcal{I},\mathcal{C},\mathcal{S}$ 。

布尔值以及 $\text{if-then-else}$ 控制流也能被编码为组合子：

<div>
$$
\begin{align}
	\mathcal{true}  & \cong \lambda x. \lambda y. x \notag \\
	\mathcal{false} & \cong \lambda x. \lambda y. y \notag \\
	\mathcal{if}    & \cong \lambda c. \lambda p. \lambda q. ((c)p)q \label{ifcomb}
\end{align}
$$
</div>

细心的读者可以发现，在组合子逻辑和$\lambda$-演算中 $\mathcal{K}$ 组合子和布尔值 $\mathcal{true}$ 都是采用的相同编码。实际上，Lisp 过程 `car` 也可以采用这样的定义。同时，布尔值 $\mathcal{false}$ 和随后将要介绍的邱奇数 $0$ 跟 Lisp 中的 `cdr` 是相同的编码。

在我们的系统中，能将 $\text{if-then-else}$ 定义为组合子，乍看之下有点奇怪，但考虑到 $\mathcal{true} \cong \text{car}$ 以及 $\mathcal{false} \cong \text{cdr}$ ，$\mathcal{if}$ 的定义就能很容易地推导出来。

<div>
$$
\begin{align}
	((\mathcal{true})P)Q & \cong \text{(car '(P Q))} \rightarrow P \notag \\
   ((\mathcal{false})P)Q & \cong \text{(cdr '(P Q))}  \rightarrow Q \notag \\
   \notag \\
   \text{if $C$ then $P$ else $Q$} & \rightarrow (((if)C)P)Q \notag \\
    & \rightarrow ((C)P)Q  \label{needabs} 
\end{align}
$$
</div>

由于式[$\ref{needabs}$]中含有自由变量，我们用三个$\lambda$-项将自由变量抽象出来，这样就得到了式[$\ref{ifcomb}$]中 $\mathcal{if}$ 的定义了。

在组合子逻辑中，*邱奇数(Church Numerals)*很好地展示了通过合适地编码，逻辑系统也可以拥有算术性质：

<div>
$$
\begin{align*}

\mathcal{0} & \cong \lambda f. \lambda x. x \\
\mathcal{1} & \cong \lambda f. \lambda x. (f)x \\
\mathcal{2} & \cong \lambda f. \lambda x. (f)(f)x \\
\dots
\end{align*}
$$
</div>

直观地来说，自然数 $n$ 在组合子逻辑中被编码为*一个将首参数在次参数上应用$n$次的函数*。邱奇数可以使用下面的组合子来执行计算：

<div>
$$
\begin{align*}
\mathcal{SUCC} & \cong \lambda n. \lambda f. \lambda x.(f)((n)f)x \\
\mathcal{PRED} & \cong \lambda n.(((n)\lambda p.\lambda z.((z)(\mathcal{SUCC})(p)\mathcal{true})(p)\mathcal{true}) \lambda z.((z)\mathcal{0})\mathcal{0})\mathcal{false} \\
\mathcal{+} & \cong \lambda m. \lambda n. \lambda f. \lambda x.((m)f)((n)f)x \\
\mathcal{*} & \cong \lambda m. \lambda n. \lambda f. (m)(n)f \\

\mathcal{ZERO?} & \cong \lambda n.((n)(\mathcal{true})\mathcal{false})\mathcal{true}
\end{align*}
$$
</div>

$\mathcal{PRED}$ 组合子乍看之下十分复杂，它的发现也颇具意味。Church 本人并没有找到 $\mathcal{PRED}$ 的定义，他试图使自己相信 $\mathcal{PRED}$ 组合子不是$\lambda$-可定义的。然而 Kleene 却发现了这个组合子[Klee81]。
