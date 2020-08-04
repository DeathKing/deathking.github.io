---
layout: post
title: "Rust蓝队须知：“内存安全”到底是什么？"
subtitle: 'Blue Team Rust: What is "Memory Safety", really?'
modified: 2020-08-03 03:42:49 -0400
tags: [rust, security, compiler, programming, exploit, pwn]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share:
disqus: y
---

+ **作　　者：** Tiemoko Ballo
+ **译　　者：** DeathKing
+ **原文地址：** [Blue Team Rust: What is "Memory Safety", really?](https://tiemoko.com/blog/blue-team-rust/)

工具影响着他们的用户和结果。C和C++的范式塑造了一代又一代的系统程序员，这两种语言的普遍性和持久力证明了它们的实用性。但由它们写就的软件数十年来却饱受[内存腐坏（Memory Corruption）](https://msrc-blog.microsoft.com/2019/07/16/a-proactive-approach-to-more-secure-code/)CVE的困扰。

作为一种[不具备垃圾回收](https://blog.discordapp.com/why-discord-is-switching-from-go-to-rust-a190bbca2b1f)的编译式语言，Rust也支持那些通常被认为是C/C++的领域。包括从高性能分布式系统到微控制器固件的所有领域。Rust提供了另一类范式，即[所有权和生存期](https://depth-first.com/articles/2020/01/27/rust-ownership-by-example/)。如果您从未尝试过Rust，请想象一下同一位近乎全知但只专注于极窄领域的完美主义者进行结对编程。这就是借用检查器（Borrow Checker）（一种实现所有权概念的编译器组件）有时会给人留下的印象。虽然带来了必要的学习曲线，但获得了内存安全保证。

与安全社区中的许多人一样，我被Rust安全替代方案的光辉前景所吸引。但是，“安全”在技术层面上实际上意味着什么？是无数飞蛾扑火的平局之一，还是Rust从根本上改变游戏？

我将根据目前为止我所学到的知识，利用这篇文章尝试回答这些问题。内存安全是深深植根于操作系统和计算机体系结构概念中的一个话题，因此，为了使本文保持简单，我必须假设读者具有必要的系统安全知识。无论你已经在使用Rust，还是对这个想法跃跃欲试，都希望本文能给你带来帮助！

<figure style="text-align: center; margin-bottom: 20px;">
  <img src="/images/post/1_btr_mem_small.jpg" alt="" class="img-margin display">
</figure>

## 从漏洞利用的角度来说，究竟什么是“内存安全保证”？

让我们从好消息开始。Rust在阻止了大部分信息泄漏和恶意代码执行的攻击向量：
+ **栈保护**：经典stack smashing攻击现在会产生一个**异常（Exception）**，而非**内存腐坏**；如果尝试越过缓冲区尾部进行写入操作，则会引发**恐慌（panic）**而不是导致**缓冲区溢出（Buffer Overflow）**。无论恐慌处理逻辑如何（例如`panic_reset`），您的应用程序仍可能遭受[拒绝服务（DoS）攻击](https://cwe.mitre.org/data/definitions/730.html)。这就是为什么针对Rust的模糊测试 [还是值得的](https://blog.hackeriet.no/fuzzing-sequoia/)。但是，由于恐慌可以防止攻击者控制的栈腐坏，因此您不会成为[任意代码执行（ACE）或远程代码执行（RCE）](https://en.wikipedia.org/wiki/Arbitrary_code_execution)的受害者。尝试越过缓冲区末尾读取同样也会停止，因此[没有类似Heartbleed的错误](https://blog.getreu.net/projects/embedded-system-security-with-Rust/)。强制是动态的：编译器在必要处插入运行期边界检查，从而只带来较小的性能开销。边界检查比C编译器可能插入的栈cookie更有效，因为边界索引在索引线性数据结构时仍然适用，这对于Rust的[迭代器API](https://doc.rust-lang.org/std/iter/index.html)来说更容易实现。
+ **堆保护**：边界检查和恐慌行为仍然适用于[堆分配](https://heap-exploitation.dhavalkapil.com/)的对象。此外，所有权范式消除了**悬挂指针（Dangling Pointers）**，从而防止了[释放后使用（UAF）](https://cwe.mitre.org/data/definitions/416.html)和使用[双重释放（DF）](https://cwe.mitre.org/data/definitions/415.html)漏洞：堆的元数据永远不会腐坏。如果程序员创建[循环引用](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html)，则仍然可能发生内存泄漏（表示永不释放分配的内存，而不是越界读取）。编译期静态分析进行**强化（enforcement）**，对代表所有可能的动态执行的抽象状态进行周全地推理。在运行期没有开销。有效性是最大化的：该程序完全不能进入错误状态。
+ **引用始终是有效的，并且变量在使用前已初始化**： safe Rust不允许操纵原始指针，以确保指针的解引用是有效的。这意味着没有DoS攻击所依赖的空指针解引用，也没有针对控制流劫持或任意读/写的指针操作。当程序员希望在逻辑上表达`NULL`这样的概念时，[`Option`类型](https://doc.rust-lang.org/std/option/enum.Option.html)就简化了错误处理。这些由所有权和生存期所带来的编译期保证。类似的编译期保证可确保变量在初始化之前无法读取。在大多数现代C编译器中，使用未初始化的变量产生的是一个警告。在这方面Rust并不新颖，但是在确保解引用的有效性上一定是的。
+ **完全消除了数据争用**： Rust的所有权系统确保任何给定的变量在任何给定的程序点只能有一个writer（例如，可变引用），而reader数量不限（例如，不可变引用）。在确保了内存安全性的同时，此方案还解决了[经典的读写并发问题](https://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem)。因此，Rust消除了数据争用，有时不需要同步原语或引用计数-但没有消除[广义数据争用](https://youtu.be/ADbCqH7_SAs?t=669)。防止数据争用减少了[并发攻击](https://www.redballoonsecurity.com/publications/papers/Concurrency_Attacks.pdf)的机会。

生活中所有美好的事物都带有限制。让我们看一下细则：

+ **并非所有的Rust代码都是内存安全的**：在Rust中实现某些数据结构（例如[双向链表](https://rust-unofficial.github.io/too-many-lists/fourth.html)），其中一项挑战就是如何满足编译器的分析。此外，某些低级操作（例如[内存映射I/O（MMIO）](https://en.wikipedia.org/wiki/Memory-mapped_I/O)）很难[完全分析](https://github.com/rust-embedded/register-rs)其安全性。标记为`unsafe`的代码块是为分析手动指定的“盲点”，因为程序员担保了其正确性，所以绕过了安全检查。这包括Rust的标准库的一部分，甚至已经为其[分配了CVE编号](https://gts3.org/2019/cve-2018-1000657.html)，并且通过扩展，也包括了通过[C外部函数接口（CFFI）](https://en.wikipedia.org/wiki/Foreign_function_interface)调用的任何外部库。此外，[研究人员发现](https://arxiv.org/abs/2003.03296)所有权的自动析构可以在`unsafe`代码中创建新的（对于Rust而言是唯一的）UAF和DF模式。可以动态检查堆一致性不变量的[强化分配器](https://www.usenix.org/system/files/conference/usenixsecurity18/sec18-silvestro.pdf)并不是完全过时的。内存安全保证的适用**范围广泛（broadly）**，但不是**普遍适用（universally）**。
+ **`unsafe`在有限的范围内放弃了内存安全性保证，但没有取消所有检查**： `unsafe`不是为所欲为的。类型，生存期和引用检查仍处于活动状态；高风险操作具有显式的API（例如get_unchecked）。尽管使用`unsafe`可能会造成内存腐坏，但这种可能性仅限于代码库的一小部分-[据估计](https://youtu.be/NQBVUjdkLAA?t=1413)，在一个典型的Rust库中比1%还要少。从安全审计的角度来看，这可以极大程度地减小主要漏洞的攻击面。可以将`unsafe`视为大型系统中的小型[可信计算库（TCB）](https://en.wikipedia.org/wiki/Trusted_computing_base)。
+ **内部可变性可以将借用检查推迟到运行期**：[内部可变性模式](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html)允许多个可变的别名引用同一个内存位置，只要它们不被同时使用。这是借用检查器的回避，是在无法以[惯用的方式重新构造](https://crates.io/crates/indextree)问题来获得强大的编译期保证时的回退。安全包装的`unsafe`的API（例如`Rc<RefCell<T>>`，`Arc<Mutex<T>>`）验证在运行期的排他性，会导致性能损失，并且可能造成恐慌。我找不到有关此模式使用与否的广泛程度地指标，但再次建议使用[模糊测试](https://rust-fuzz.github.io/book/afl/tutorial.html)来进行概率性恐慌检测。

老实说，数十年来硬件、操作系统和编译器级别的防御措施已经使C和C++部署得到了加强。内存腐坏的0-day漏洞并不是一件容易的事。然而，Rust仍然感觉像是向前迈出了重要的一步，并且在性能攸关型软件的安全性方面有了显著改进。即使`unsafe`逃生出口必须存在，但也可以很大程度上消除内存腐坏（一大类严重的BUG）。

那么，Rust是新的救世主，被派来救我们脱离远程Shell的地狱吗？当然不是。Rust不会阻止命令注入（例如，输入字符串的一部分最终作为`execve`的参数）。或不当配置（例如，回退到不安全的密码）。又或者或逻辑错误（例如，忘记验证用户权限）。任何通用的编程语言都不会使您的代码具有内在的安全性或形式上正确。但是至少您不必担心这些错误，也不用在整个Rust代码库中维护[复杂而又不可见的内存不变量](https://alexgaynor.net/2019/apr/21/modern-c++-wont-save-us/)。

## 好吧，嵌入式系统呢？它们不是更加脆弱吗？

假设“嵌入式”表示没有操作系统抽象；软件栈是单体式二进制文件（例如AVR或Cortex-M固件）或操作系统本身的一部分（例如内核或引导程序）。Rust的[`!#[no_std]`属性](http://cliffle.com/blog/m4vga-in-rust/#on-no-std)有助于开发嵌入式平台。`!#[no_std]`Rust库通常会放弃动态集合（如Vec和HashMap），以实现对裸机环境（没有内存分配器，没有堆）的可移植性。没有了动态内存借用检查的屏障是极小的，因此原型开发便利度仍大致相当于嵌入式C——尽管有[较少的](https://forge.rust-lang.org/release/platform-support.html)支持[架构](https://gcc.gnu.org/install/specific.html)。

资源受限和/或实时嵌入式系统`[!#no_std]`目标通常缺乏现代的缓解措施，例如[内存保护单元（MPU），No eXecute（NX）或地址空间布局随机化（ASLR）](https://www.youtube.com/watch?v=Dvug1Hup2iA)。我们正在谈论的是一片法外之地，那里的内存布局是平坦的，没有人能听到您的段错误。但是，Rust仍然为我们提供了在没有分配器的情况下运行裸机的那种甜美的边界检查保险。值得注意的是，它可能是嵌入式方案中的第一道和最后一道防线。请记住，与硬件的低级交互可能需要一定数量的unsafe代码，其中就有不带边界检查的内存访问。

对于x86/x64，Rust编译器还会插入栈探针以检测栈溢出。目前，此功能不适用于`!#[no_std]`或其他体系结构-尽管已提出了[创造性的链接解决方案](https://blog.japaric.io/stack-overflow-protection/)。栈探针通常通过保护页实现，可防止由于无休止的递归而耗尽栈空间。另一方面，边界检查可防止基于栈或基于堆的缓冲区溢出错误。这是一个微妙而又重要的区别：对于安全性，我们通常更关心后者。

请记住，从Rust的角度来看，内存是一种软件抽象。当抽象终结时，保证也将无效。如果物理攻击（侧信道攻击，故障注入，芯片解封装等）是威胁模型的一部分，则没有理由相信语言选择可以提供任何保护。如果您忘了烧写适当的锁定位，并在其中暴露了调试端口，并且用于EEPROM解密/身份验证的对称密钥位于[EEPROM中](https://en.wikipedia.org/wiki/EEPROM)：现场的攻击者将不需要内存腐坏漏洞。

## 太酷了，但是日常开发又如何呢？

依赖管理并不如有着花哨营销名称的漏洞利用或用来防止它们的新时代的编译器分析技术那样迷人。但是，如果您曾经负责生产基础架构，那么你就会知道补丁延迟通常是最重要的指标。有时是你的代码遭到了破坏，但更多时候是依赖的库使你的系统处于危险之中。这是Rust的软件包管理器cargo发挥着不可估量的作用的地方。

cargo启用可组合性：你的项目可以将第三方库集成为静态链接的依赖项，并在首次构建时从集中式存储库下载其源代码。它使依赖关系维护更加容易-包括将最新的补丁程序（安全性或其他方面的信息）拉入到您的版本中。C或C++生态系统中没有类似物可以提供cargo的[语义版本控制](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html)，但是管理一组[git子模块](https://git-scm.com/book/en/v2/Git-Tools-Submodules)可能会产生类似的效果。

与C/C++子模块的Duck Tape不同，上述可组合性在Rust中是内存安全的。C/C++库间传递结构指针，而没有强制约定由谁执行清除任务：你的代码可能会释放库中已释放的对象-这是一个可能的错误-会产生新的DF错误。Rust的所有权模型提供了契约，简化了各个API之间的互操作性。

最后，由于cargo提供了第一级测试支持，现代C和C++经常会遗漏的组件。Rust的工具链使软件工程的工程部分更加容易：测试和维护非常简单。在现实世界中，这对于总体安全状况与内存安全同样重要。

## 等等...难道我们不会忘记整数溢出吗？

不完全是。整数溢出并不算是内存安全问题，但它肯定是造成任意命令执行（ACE）的一条复杂内存腐坏BUG链的一部分。假设一个有问题的整数在写入攻击者控制的数据之前被用作索引，安全的Rust仍会阻止该写入。

无论如何，整数溢出会导致令人讨厌的错误。 cargo使用可[配置的](https://doc.rust-lang.org/cargo/reference/profiles.html)构建配置文件来控制编译设置，其中就包括整数溢出处理。默认`debug`（低优化）配置文件包括`overflow-checks = true`，因此二进制输出会在开发人员没有显式地处理整数溢出（例如，`u32::wrapping_add`）时恐慌。除非配置被覆写，否则`release`（高优化）模式将执行相反的操作：允许静默包装（wrap-around），就像C/C++，因为删除检查会提高性能。与C/C++不同，[Rust中整数溢出不是未定义的行为](http://huonw.github.io/blog/2016/04/myths-and-legends-about-integer-overflow-in-rust/)；可以认为Rust中封装后的二进制补码是可靠的。

如果性能是第一要务，则测试用例应该在`debug`构建下争取足够的覆盖率，以捕获大部分整数溢出。如果安全性是第一要务，请考虑在`release`模式下启用溢出检查并试图引发恐慌。

## 结论

内存安全并不是一个新主意，垃圾回收和智能指针已经存在了一段时间。但有时正是一个**正确的**实现使得一个现有的**好**想法变成一个新颖的**伟大**想法。Rust实现了[仿射类型系统](https://en.wikipedia.org/wiki/Substructural_type_system#Affine_type_systems)的所有权范式就是个伟大想法，可以在不牺牲可预测性能的情况下实现安全性。

现在，我(卑鄙地)追求务实，而不是教条。对于生产嵌入式项目，完全有理由坚持使用成熟的供应商[HAL](https://en.wikipedia.org/wiki/Hardware_abstraction#In_operating_systems)和C工具链。许多现有的C / C ++代码库应该被模糊测试、加固以及维护——而不是用Rust重写。一些库绑定（例如[z3求解器的](https://ericpony.github.io/z3py-tutorial/guide-examples.htm)库绑定就是一个示例）从动态解释语言的类型中受益匪浅。在某些领域，具有[Hoare逻辑](https://en.wikipedia.org/wiki/Hoare_logic)前置条件和后置条件的语言可能证明生产力受到打击（例如[Spark Ada](https://en.wikipedia.org/wiki/SPARK_(programming_language))）。物理攻击通常与语言无关。**总结：没有工具是万能药**。

除了免责声明外，我不记得上一个使我停下脚步并像Rust一样注意到它的新技术。该语言将编译器本身的系统编程最佳实践具体化，以开发时的心智负担（cognitive load）换取运行期的正确性。对于可能会给内存安全带来风险的模式，也提供了必要的显式选择（例如，`unsafe`、`RefCell<T>`）。对主要类别BUG的减缓像是合法性的左移测试（legitimate shift left）：可利用漏洞的显著子集变成了编译期错误和运行期异常。 [Ferris](https://rustacean.net/)的外壳相当坚硬。
<hr>

## 关于术语

计算机术语的翻译通常充满争议。每个译者都因极力追求译文的“信、达、雅”。本文中的几处术语，我并没有采用常见的译法。我将陈述其中的缘由，还望各位读者批评指正：

1. **内存腐坏（Memory Corruption）**，常译为“内存损坏”。然而memory一词不单可以指抽象意义上的“计算结果存储空间”，也可以指实在的内存条。当memory corruption被翻译为“内存损坏”时，容易使人联想到由于内存条的元件发生故障而产生电子错误、物理上的损坏。而就造成memory corruption的具体过程来说，通常是由外部数据恶意造成越界的内存读写，从而破坏（corrupt）了内存的完整性（integrity）约束——这很容易使人联想到腐败（corruption）损害了人的正直（integrity）。“损坏”通常导致的是“不可用”，而“腐坏”则只是“不可信”、“不可靠”。综合上述两个方面，我更倾向于使用“腐坏”来描述“变质”这个过程。
2. **编译期（Compile-time）、运行期（Runtime）**，也译作编译时、运行时。将runtime翻译为运行时，通常也指代运行时环境，如Java运行时（环境）、Golang运行时（环境）。这里为了明确这两个术语的时间属性，故选择将其翻译为“某某期”。