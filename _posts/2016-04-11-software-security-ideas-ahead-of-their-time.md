---
layout: post
title: "那些超前时代的安全理念"
subtitle: "Software Security Ideas ahead of Their Time"
modified: 2016-04-11 19:57:25 +0800
tags: [ASLR, bounds checking, buffer overflow, CFI, randomization, software diversity, stack canary]
image:
  feature: 
  credit: 
  creditlink: 
comments: y
share: 
disqus: y
---

+ **作　　者：** Michael Hicks, Andrew Ruef  
+ **译　　者：** DeathKing, 樊昱才, 曹鹤梅  
+ **原文地址：** [Software Security Ideas Ahead of Their Time](http://www.pl-enthusiast.net/2016/02/01/software-security-ideas-ahead-of-their-time/)

作为研究者，我们经常被要求从水晶球中预测未来。我们试图预测未来将面临的问题，这样一来，当问题实际到来之时，我们当前开始做的工作就能够帮助我们解决掉它。有时候，研究人员会去猜测将要出现的问题以及可能的解决方案，但是却不选择去追寻到底。某种意义上来说，她发现了一个超前于时代的想法，但却又抛弃了它。

最近，Andrew 的一个朋友使他注意到了20年前的一封在“防火墙”邮件列表中的邮件，这封邮件漫不经心地揭示却又抛弃了一些现在相当重要的问题，以及它们的解决方案，并且这些问题处于软件安全的前沿。这样的情形不仅很戏剧性，并且也有指导意义，特别是这些想法跟程序设计语言研究领域紧密相关，但是（就我们所知）这些并没有被当时的程序设计语言研究人员考虑在内。

## 问题：利用缓冲区溢出的攻击

在“防火墙”邮件列表中的这封写于1995年的交流信件里，人们明确提出了基于缓冲区溢出的攻击：

> 花时间来挖掘一个针对特定目标平台，操作系统版本和CPU类型的漏洞是可行的。这个攻击在不同版本或机器类型上可能需要不同的**偏移量（offset）**，并且这明显的是一个机械的过程。  
> 这种（利用）过程慢慢变得乏味。

这段充满倦意的陈述在[Smashing the Stack for Fun and Profit](http://insecure.org/stf/smashstack.html)这篇文章发表的一年前做出，而它里程碑式地系统描述了堆栈破坏攻击。这段陈述同时也预言了[自动发现和挖掘漏洞的研究](https://www.isoc.org/isoc/conferences/ndss/11/pdf/5_5.pdf)。作者继续道：

> 我只是觉得这样会变得非常有趣：如果我们有一个C语言的编译器，对于每一次编译，它都可以打乱堆栈中变量的位置，而且它还可以任意地在堆栈上很多位置中申请一些没有赋值的短整型变量，比如说要调用 `sprintf`，`strcpy` 等函数的长字符串后。  
> 在未来10年里我们需要一些保护机制来抵抗频繁的堆栈攻击，提高所有防御机制的水平。虽然这并不会提高攻击者第一次攻击代价，但它可以抵御那些通过大量攻击获利的人。

## 漫不经心的解决方法：20年的研究

令人惊奇的是邮件中这些想法基本上都是半开玩笑的被提出，但在之后20年竟成为学术上正式的话题，甚至在某些领域成为主流的发展。

### 栈随机化（2010-当今的研究）

原始邮件中引述了有关一种可能防御措施——栈布局随机化。但是邮件列表中的一些人打消了这个念头（这之后更多人也放弃了），但是截至2010年，[Michael Franz](http://www.michaelfranz.com/)主张应该[重提这个想法](http://dl.acm.org/authorize.cfm?key=312386)。最近5年中他的团队和其他学者在[自动化软件多样性](https://www.ics.uci.edu/~perl/automated_software_diversity.pdf)方向做了引人入胜的事情，当然也包括栈布局随机化。

### 安全栈（2006-现已成为编译器扩展）

列表中的其他人回复了他们疯狂的想法，比如用[两个栈](http://www.greatcircle.com/firewalls/mhonarc/firewalls.199508/msg00836.html)：

> 既然我们在讨论“不切实际(Blue-sky)”的解决方案，那还为什么还畏首畏尾的呢？大家都知道确保代码**完整性（Integrity）**的方法就是将其和数据分开，一个栈存放用户数据和程序状态信息（函数返回地址等）并没有保持这种分离。解决方法就是使用两个栈，一个仅用来存放返回地址和程序**上下文(Context)**信息，另一个放用户数据。

在2006年，[XFI](https://www.usenix.org/legacy/event/osdi06/tech/full_papers/erlingsson/erlingsson.pdf) 系统详尽的探索了双栈模式（XFI 也提供其他保护）。这个概念目前已经成熟的运用在一种产品级 [LLVM](https://llvm.org/) 前端—— [clang](http://clang.llvm.org/) 编译器上，比如就像 [SafeStack](http://clang.llvm.org/docs/SafeStack.html)。

### 边界检查（1955 - 成为主流）

提出**分段栈**的作者同时也在第一时间提出了需要阻止越界存取（access）的思想：

> 另一种解决方法是给编译器加上 Pascal 式的**运行时边界检查**功能。当然，当数组边界异常发生之后，你需要去解决这个问题。以 Pascal 为例，默认的执行步骤是，宣告异常，并且终止该程序（这难道不会为‘拒绝服务’攻击创造极佳的条件吗？！！）。你防止这种终止发生的方式就是在一开始进行自我检查，并且永远不要对一个数组的末尾之后进行访问。这样，你就对所有的情形都进行了双重检查。

[Jones 和 Kelly ](https://www.doc.ic.ac.uk/~phjk/BoundsChecking.html)在那同一年（1995）提出了对 C 语言的向后兼容的边界检查，同样的概念也存在于类型安全（且执行边界检查）的[ Java 语言](http://www.oracle.com/technetwork/java/javase/overview/javahistory-index-198355.html)中。Jones 和 Kelly 的方法十分慢以至于无法应用于实践，但是其优点被认为具有足够好的前景，并且在此之后，涌现出了一长串有关于具有类型安全（或者[内存安全](http://www.pl-enthusiast.net/2014/07/21/memory-safety/)）特性的 C 的研究，包括例如[ CCured ](https://www.cs.virginia.edu/~weimer/p/p477-necula.pdf)和[ Cyclone ](https://www.cs.virginia.edu/~weimer/p/p477-necula.pdf)在内的20世纪早期的语言，以及在 2009-2010 年期间出现的[ Softbound/CETS ](https://www.cs.rutgers.edu/~santosh.nagarakatte/softbound/)。运行时边界检查的硬件支持（基于[ Nagarakatte ](https://www.cs.rutgers.edu/~santosh.nagarakatte/)以及其他参与 Softbound 的人员的工作）被作为 Intel ISA 的扩展（称为[ MPX ](https://en.wikipedia.org/wiki/Intel_MPX)）而发布。

### 栈金丝雀（1998 - 当今主流做法）

其他的作者后来又即兴想出了[一些想法](http://www.greatcircle.com/firewalls/mhonarc/firewalls.199508/msg00815.html)，建议我们使用现在被称为**栈金丝雀（Canary）**的方法：

> 我给你一个更好的解决方法，把一个整形或类似类型的变量正正好分配到缓冲区后，那里的值被编译器随机设置并且在执行读取命令后检查。如果缓冲区被重写，金丝雀便很难和它刚写入时的值相匹配。这并不会阻止机器使用者重新编译，也不需要选择一个特别的值作为金丝雀，但是这会阻止大多数的攻击者。

Cowan 等人在1998年发表的[文章](https://www.usenix.org/legacy/publications/library/proceedings/sec98/full_papers/cowan/cowan.pdf)丰富了栈金丝雀理论。这个理论随后有一些改善：没有使用静态确定的整数，转而使用了一个动态分配的，这个都在文章发表之后不久被添加进了主流编译器（原始论文展示这个功能的一个 GCC 补丁）。

### 一些没有被提及的想法

“防火墙”讨论帖的作者们即使有着令人难以置信的先见之明，但是他们仍然忽略了一些防范缓冲区溢出渗透的重要手段。他们并没有明确出在2001年首次提出的地址空间[布局随机化（ASLR）](https://en.wikipedia.org/wiki/Address_space_layout_randomization)的想法，尽管他们对于栈随机化的想法的概念基本一致。他们也没有明确在2005年首次提出的[控制流完整性（CFI）](http://research.microsoft.com/pubs/64250/ccs05.pdf)的想法。CFI （和提出的其他想法一样）采用不同的方式编译程序，以限制单次攻击可能造成的影响。寻找到在精确度和性能方面的正确平衡也是 CFI 的一个研究课题，但是这将它的方向引入到了主流研究当中。过去20年间更多被探索的问题在2013年 Szekeres 等人的论文中被非常漂亮地说明了—— [《SOK: Eternal War in Memory》](https://www.cs.berkeley.edu/~dawnsong/papers/Oakland13-SoK-CR.pdf)。

## 怎么样以及为什么

查阅这份邮件交流后，我们也许会发问：自1995年以来，什么发生了改变？并且改进了那些不切实际（至少在那些作者看来）的想法。

1995年的人们[尤其关心编译器的复杂化——这是可信计算的基石](http://www.greatcircle.com/firewalls/mhonarc/firewalls.199508/msg00861.html)。

> 但是当有人建议通过打开并修改**其它的**大型代码生成程序来解决这些问题，我的忧虑便突然爆发。或许是我脾气古怪又生性多疑，但我确信，“没有专家充分审查编译器，并断定它们在所有的平台上都是正常且安全的，而只是像上述那样通过修改编译器才是走向安全且可靠的进步之道”，这样的信条我是绝不接受的。

这种担心是相当合情合理的。在过去的20年中，无论是编译器构造的技艺（art）还是实践都进步不小。现在有一种被称为[ CompCert ](http://compcert.inria.fr/)的产品级编译器，其正确性已经被形式化地验证过了；给这个编译器写扩展并不会造成太多问题。此外，[LLVM ](http://llvm.org/)编译器的出现，提供了一种健壮（robust）的模块化基础设施，使得我们能够在其基础上开发令人激动的编译器扩展，这使得我们对所开发扩展的正确性抱有极大的信心, 并且也加速了采用的过程。[ Vellvm ](http://www.cis.upenn.edu/~stevez/vellvm/)项目似乎也在为 LLVM 的一些部分添加验证，这也可以消除“忧虑警报”。

似乎在讨论“运行时检查是昂贵且古怪的，为什么不在一开始就让代码正确呢？”（这也是某些[计算机科学单口喜剧](https://vimeo.com/95066828)的笑点所在）这个问题时有一种“潜在情绪（undercurrent）”。或许，“第一次就把 C 语言代码写正确”这种想法存在了相当长一段时间，人们终于被残酷的现实打败——第一次就把 C 语言代码写正确是相当之困难的。也许我们需要开始思考一些远比邮件列表中提到的那些玩笑还要有用的运行时防护。


### 为什么在“防火墙”邮件组进行讨论？

我们会思考的另一个问题是：为什么这些想法是出现在讨论关于“防火墙”的邮件列表上，而非程序设计语言社区？毕竟，这些提议跟语言和编译器设计密切相关。关闭这个讨论帖的作者也想搞清楚这个问题：

> 依我所见，所有的这些讨论都跟**防火墙**偏离太多。更进一步的讨论应该在`comp.lang.*` 新闻组中发布。

一种可能是因为在实际应用中防火墙社区比程序设计语言社区更进一步接触漏洞利用攻击：他们能够识别出攻击依赖于哪种系统属性，并且他们能够在高层次想出，在保持系统原有语义的情况下，如何修改系统可以打破攻击所利用的依赖关系。

现在，相当多的从事次时代运行时安全系统的开发人员都在同那顶尖安全专家合作——后者深谙现代漏洞攻击之道。确实，就如 Mike 之前所写那样，当我们把程序设计语言中的想法应用在其它的领域提出的问题时，比如[密码学](http://www.pl-enthusiast.net/2014/12/17/synergy-programming-languages-cryptography/)或[机器学习](http://www.pl-enthusiast.net/2014/09/08/probabilistic-programming/)，将会产生强大的协同增效作用。

## 未来

通过阅读本文，你可能开始沉思，哪种技术在当下看来像是不可能的长射门，但实际上在会20年内最终会变得有重要价值。我们想在评论区中看到您的想法。

这里有几个我们能想到的最靠谱的预测：

+ **验证剩余部分**。因为很多原因，进行验证是像“白日梦”一样：验证工具即慢又难于使用，思考如何表述你想证明的性质可能会很困难，并且，证明和实现之间总是有着间隙。然而，像[ frama-c ](http://frama-c.com/)、[F*](https://www.fstar-lang.org/)、[bedrock](http://plv.csail.mit.edu/bedrock/)以及[dafny](http://research.microsoft.com/en-us/projects/dafny/)这样的系统似乎都缩窄了关于可用性的间隙，这些都是通过使证明与实现高度相关来达到的。诸如[IronClad](https://github.com/Microsoft/Ironclad)和[Sel4](https://sel4.systems/)这样的项目以及[HACMS](http://www.darpa.mil/program/high-assurance-cyber-military-systems)基金，都在推动当今最先进技术的发展。
+ **代码注入攻击将终结**。我们受尽了低层次安全漏洞的折磨，这些漏洞通常允许敌手将一整个程序注入到一个有漏洞的程序中，文献上通常称之为缓冲区溢出（Buffer overflows）。这里提到的技术已经关上了代码注入攻击的大门，比如，有报道指出，[近来来基于栈的攻击已经淡出主流](https://blogs.microsoft.com/cybertrust/2014/06/24/how-vulnerabilities-are-exploited-the-root-causes-of-exploited-remote-code-execution-cves/)。这个趋势还会继续。这并没有终结安全议题，但是未来20年的关注点将与现在相当不同。
+ **信息泄露还将继续**。比证明C语言程序的基本的安全性质难得多的问题便是证明程序控制、释出由它们所掌控的信息的性质，比如（一种变体）[noninterference](http://www.pl-enthusiast.net/2015/03/03/noninterference/)。基于时间和资源消耗的信息泄露没有引起研究者的太大关注（这也促使了最近的 DARPA [STAC](http://www.darpa.mil/program/space-time-analysis-for-cybersecurity)项目）。如果继续采用云计算技术，攻击者实际期望的是小秘密或者密钥的泄露。

