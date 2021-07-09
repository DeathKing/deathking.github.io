---
layout: post
title: "编译Rust是NP困难的"
subtitle: 'Compiling Rust is NP-hard'
modified: 2021-07-08 22:42:00 -0400
tags: [rust, compiler, programming, complext, sat]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share:
disqus: y
---

+ **作　　者：** NieDżejkob
+ **译　　者：** DeathKing
+ **原文地址：** [Compiling Rust is NP-hard](https://niedzejkob.p4.team/rust-np/)，译文略有删节。


...虽然不是旗舰借阅检查有问题。我注意到并且今天想与大家分享的是，Rust 编译器对match模式执行的详尽检查是SAT问题的超集 。

尽管不是借用检查这个旗舰特性的过错……但我今天想与你分享的是，我注意到由 Rust 编译器所进行的穷尽性检查（exhaustiveness checking）是 SAT 问题的一个超集（superset）。

## 穷尽性检查

考虑以下代码（[playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=cab7e0bcf26b0180f15324d81009870d)）：

```rust
fn update_thing(old_thing: Option<Thing>, new_thing: Option<Thing>) {
    match (old_thing, new_thing) {
        (None, Some(new)) => create_thing(new),
        (Some(old), None) => delete_thing(old),
        (None, None) => { /* nothing to be done */ },
    }
}
```

正如大多数 Rustaceans 已经知道的那样，编译器将拒绝此操作并显示以下错误：

```rust
error[E0004]: non-exhaustive patterns: `(Some(_), Some(_))` not covered
 --> example.rs:4:11
  |
4 |     match (old_thing, new_thing) {
  |           ^^^^^^^^^^^^^^^^^^^^^^ pattern `(Some(_), Some(_))` not covered
  |
  = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
  = note: the matched value is of type `(Option<Thing>, Option<Thing>)`
```

如果你像我一样，你迟早会开始想知道这实际上是如何计算的。实现这个，你可以编写的最有效的算法是什么？事实证明，在最坏的情况下你不能做得很好——我们将通过从 SAT 归约证明这一点。

## 布尔可满足性

布尔可满足性，或简称 SAT，是给定一个布尔公式，确定是否有一种方法可以将 1 和 0 分配给其中的变量，使得公式的计算结果为 1。例如，公式

```rust
A and (A xor B)
```

是可满足的，只需要使 `A = 1` 并且使 `B = 0`，但是

```rust
A and B and (A xor B)
```

不是可满足的。

这种公式的标准表示形式是合取范式（conjunctive normal form），或简称为：AND-of-ORs（译注，即一系列析取子句的合取）。转换后，我们之前的示例将如下所示：

```rust
A and B and (A or B) and (!A or !B)
```

这通常写成由常量构成的列表——我们需要从其中每一行中选择一个选项而不引起冲突：

```rust
 A
    B
 A  B
!A !B
```

显然，每个公式都可以转化为这种形式。在不造成指数爆发的情况下做到这一点有点困难，但我们不需要关心细节。此时，这种表示应该类似于一组 Rust 模式。

## 把两者联系起来

想明白穷尽性检查和可满足性之间的等价性有点棘手，因为在推理的每一步，它们之间都有一个否定：如果一条 `match` 表达式被拒绝，`rustc` 会向我们展示一个我们没有涵盖的示例值，同时，这就是我们 SAT 问题的解。因此，与任何给定模式相匹配的值都会被拒绝。这意味着我们可以像这样对上一节中的示例进行编码：

```rust
match todo!() {
    (false, _)     => {}  //  A
    (_,     false) => {}  //     B
    (false, false) => {}  //  A  B
    (true,  true)  => {}  // !A !B
}
```

让我们仔细看看最后一个模式，`(true, true)`。因为这条模式，任何未涵盖的值必须包含一个 `false`。这正是我们的`!A !B`子句所指定的。

> 译注：  
> 原文出于行文的简洁，似乎没有很直白地阐明两者的关系。首先请注意 `match` 的每一条模式（pattern），与合取范式的每一个子句（clause）都是取反的关系。也就是说，如果 `match` 表达式成功被匹配，那么成功匹配的那条模式所对应的子句必然赋值为 `false`，由于合取范式各子句间都是通过逻辑且连接，那么合取范式整体必为 `false`。  
> 反之，如果合取范式是可满足的，那么说明存在着某个值，使得 `match` 表达式被拒绝——这就是 `rustc` 需要找到的那个反例。

## 基准测试

注意到这一点后，我无法不将 rustc 与其他 SAT 求解器比较。SAT 问题的标准文件格式是[DIMACS CNF](https://logic.pdmi.ras.ru/~basolver/dimacs.html)，多亏了 Jannis Harder 的[flussab_cnf](https://crates.io/crates/flussab-cnf)库，处理它变得轻而易举。转换器的核心看起来像这样：

```rust
/// 0-based variable index, possibly negated — `false` in the `bool` field means negated
#[derive(Clone, Copy, PartialEq, Eq)]
struct Literal(usize, bool);

fn print_clause(mut clause: Vec<Literal>, num_variables: usize) {
    let mut pattern = vec![None; num_variables];
    for Literal(var, positive) in clause {
        // We negate it here, as we need to match the assignments that *don't* satisfy
        // the clause. While not doing this would generate an equivalent instance, this
        // way the results rustc outputs directly correspond with our input.
        pattern[var] = Some(!positive);
    }

    let pattern = pattern.into_iter()
        .map(|pat| match pat {
            None => "_",
            Some(true) => "true",
            Some(false) => "false",
        })
        .join(", ");

    println!("({}) => {{}}", pattern);
}
```

如果你想自己运行一些实验，我已经将[完整代码](https://github.com/NieDzejkob/rustc-sat)推送到 GitHub。我选择来自于[SATLIB 基准测试问题集](https://www.cs.ubc.ca/~hoos/SATLIB/benchm.html)`uf20-91` 的第一个实例上尝试它。这是一个随机生成的问题，有 20 个变量和 91 个子句。这并不难——即使是我对基本[DPLL](https://en.wikipedia.org/wiki/DPLL_algorithm)算法的简单实现也能在不到一毫秒的时间内解决它。

`rustc` 表现又如何呢？它花了 15 秒，并产生以下输出：

```rust
warning: unreachable pattern
  --> test.rs:34:1
   |
34 | (_, _, _, _, _, _, true, _, _, _, _, false, _, true, _, _, _, _, _, _) => {}
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: `#[warn(unreachable_patterns)]` on by default

warning: unreachable pattern
  --> test.rs:55:1
   |
55 | (_, _, _, _, true, _, _, true, _, _, _, false, _, _, _, _, _, _, _, _) => {}
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

[...]

warning: unreachable pattern
  --> test.rs:92:1
   |
92 | (_, _, _, false, true, _, _, _, _, _, _, _, _, _, _, true, _, _, _, _) => {}
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

error[E0004]: non-exhaustive patterns: `(false, true, true, true, false, false, false, true, true, true, true, false, false, true, true, false, true, true, true, true)`, `(true, false, false, false, false, true, false, false, false, false, false, false, true, true, true, false, true, false, false, true)`, `(true, false, false, false, false, true, false, false, true, false, false, false, false, true, true, false, true, false, false, true)` and 4 more not covered
 --> test.rs:1:19
  |
1 | fn main() { match todo!() {
  |                   ^^^^^^^ patterns `(false, true, true, true, false, false, false, true, true, true, true, false, false, true, true, false, true, true, true, true)`, `(true, false, false, false, false, true, false, false, false, false, false, false, true, true, true, false, true, false, false, true)`, `(true, false, false, false, false, true, false, false, true, false, false, false, false, true, true, false, true, false, false, true)` and 4 more not covered
  |
  = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
  = note: the matched value is of type `(bool, bool, bool, bool, bool, bool, bool, bool, bool, bool, bool, bool, bool, bool, bool, bool, bool, bool, bool, bool)`
  = note: this error originates in a macro (in Nightly builds, run with -Z macro-backtrace for more info)

error: aborting due to previous error; 17 warnings emitted

For more information about this error, try `rustc --explain E0004`.
```

与基本的SAT求解器不同，它找出了所有的解决方案以及一些重复的子句。尽管有这些特性，我不能说它在性能上具有竞争力。

## 改进表示方式

如果你观察我们生成的 Rust 代码，你会注意到它主要由 `_` 构成。事实上，它的大小是输入SAT问题规模的平方（因为变量和子句的数量对于输入规模来说都是线性的）。虽然这仍然是一个多项式，因此可以证明它是 NP 困难的是可行，但它看起来并不是最佳的。

我们可以用树型结构来替代简单的列表：

```rust
  (bool, bool,   bool, bool,     bool, bool,   bool, bool)
(((bool, bool), (bool, bool)), ((bool, bool), (bool, bool)))
```


这样，如果一个分支只由 `_` 组成，我们就可以将它们全部折叠成一个 `_`。这使得单个句子只接受 `O(|子句中的变量|*log(|总变量数|))` 个字符，这意味着整体复杂度关于输入大小是线性的。

## 结束语

这是否意味着 `rustc` 应该集成一个工业级的 SAT 求解器？尽管那会很有趣，但我不提倡那么做。在面对一个由无聊的书呆子构造的病态示例时，它才会有性能问题，我认为不应该在此上花费宝贵的工程时间。此外，泛化一个 SAT 算法来处理 Rust 模式的全类型表达，借用数学家的一些语言，可能是非平(non-trivial)的。

最后，SAT 是一个令人惊讶的深度和有趣的领域。尽管这是一个 NP 完全问题，但现代求解器可以处理具有数千个变量的实际问题。如果您想了解更多信息，我可以推荐 [Knuth 关于该主题的讲座](https://www.youtube.com/watch?v=g4lhrVPDUG0)，以及 [Jannis 关于 Varisat 的博客系列](https://jix.one/blog/)。[The Handbook of Satisfiability](https://ebooks.iospress.nl/volume/handbook-of-satisfiability-second-edition) 也可以作为参考。


