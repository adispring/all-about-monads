# All_About_Monads 中文

All_About_Monads 原文地址：https://wiki.haskell.org/All_About_Monads

本教程旨在以一种易于理解的方式对Monad的概念及其在函数式编程中的应用进行说明，这对初学者和中级Haskell程序员都是有用的。假定您熟悉Haskell语言，但无需具备使用Monad的经验。本教程涵盖了许多材料，后面的部分则需要对以前的材料有透彻的了解。在演示monadic编程的过程中提供了许多代码示例。本文还是有一些难度的，不要企图一遍就能理解所有的内容。

本教程分为三个部分。第一部分提供了有关Monad在函数式编程中的作用，Monad如何运行以及在Haskell中如何声明和使用Monad的基本知识。第二部分介绍了Haskell中的每个标准monad，给出了monad的定义并讨论了monad的用法。第三部分介绍了与monad transformers 有关的高级材料以及在使用monad进行编程时遇到的实际问题。

真正理解monad的最好方法是不断尝试monadic代码。本教程提供了许多代码示例，范围从简单到中等复杂，建议您学习，运行它们，并能尝试修改它们。前面的示例可能会读起来比较生硬、不自然，这是为了以避免过多 monad 同时出现，造成理解上的困难。 。如果您在尝试示例代码时发现自己对此限制感到沮丧，可以阅读第三部分，了解如何克服该限制。

## 内容

### 第一部分 - 理解 Monads

* 引言
  * 什么是 Monad？
  * 为什么我应该努力搞懂 monads？
  
* 初次接触 Monads
  * 类型构造器（Type constructors）
  * Maybe a monad
  * 一个例子
  * List 也是一个 monad
  * 总结
  
* 使用 class
  * Haskell 类型类
  * Monad 类
  * 继续上面的例子
  * Do 标记
  * 总结
  
* Monad 的规则
  * 三个基本的规则
  * Failure 是一种选择
  * 没有出路
  * 零和加法
  * 总结

* 练习

* Haskell 中的 Monad 支持
  * 在标准的 prelude 中
  * 在 Monad 模块中
  * 总结

### 第二部分 - 一系列标准 Monads

* 引言

* Identity monad
  * 概述
  * 目的
  * 定义
  * 示例
  
* Maybe monad
  * 概述
  * 目的
  * 定义
  * 示例

* Error monad
  * 概述
  * 目的
  * 定义
  * 示例

* List monad
  * 概述
  * 目的
  * 定义
  * 示例

* IO monad
  * 概述
  * 目的
  * 定义
  * 示例

* State monad
  * 概述
  * 目的
  * 定义
  * 示例

* Reader monad
  * 概述
  * 目的
  * 定义
  * 示例

* Writer monad
  * 概述
  * 目的
  * 定义
  * 示例

* Continuation monad
  * 概述
  * 目的
  * 定义
  * 示例

### 第三部分 - 现实世界中的 Monad

* 引言

* 组合 monads 时存在的疼点
  * 嵌套的 Monads
  * 组合的 Monads

* Monad transformer

* Monad type 构造器

* 标准 monad transformers
  * MonadTrans 和 MonadIO classes
  * 标准 monads 的 各种 Transformer 版本
  
* 剖析一个 monad transformer
  * 组合的 monad 的定义
  * 定义 lifting 函数
  * Functors(函子)

* 更多 Monad transformers 示例
  * 带 IO 的 WriterT
  * 带 IO 的 ReaderT
  * 带 List 的 StateT

* 管理 transformer 栈
  * 选择正确的顺序
  * 多个 transformers 的例子
  * 重的 lifting （提升）

* 继续探索 

### 附录

* 附录 I - Monad 的物理类比

* 附录 II - Haskell 代码示例

## 引言

  * 什么是 Monad？
  * 为什么我应该尽力搞懂 monads？

### 什么是 Monad ？

一个 monad 是一种根据值和使用这些值的计算序列来构造计算的方法。 Monads 允许程序员使用顺序构建的模块块来构建计算，顺序构建的模块本身也可以是计算序列。monad 决定如何组合这些计算形成新的计算，使程序员在需要时不必手动编写这些组合。

monad 可以看作是：将简单的计算组合为更复杂的计算，这是非常有用的。例如，您应该熟悉 Haskell 中的 `Maybe` type：

```hs
data Maybe a = Nothing | Just a
```

代表可能无法返回结果的计算类型。 `Maybe` type 表示一种合并 “返回 Maybe 值” 的多个计算 的策略：如果合并的计算由 计算 A 和 计算 B 组成，且计算 B 依赖于计算 A 的结果，则只要 A 或 B 任意一个产出 `Nothing` ，则合并的计算应该产生 `Nothing`，当两个计算都成功时，合并计算的结果等于 计算 B 应用 计算 A 的结果而得到的输出。

其他 monad 可以用于构建执行 I/O，保存状态，可能返回多个结果等的计算。就像有各种组合计算的策略一样，存在各种类型的 monad，但是有些 monads 特别有用，常见到足以使它们成为标准 **Haskell 98 libraries** 的一部分。这些 monads 会在第二部分中分别介绍。 

### 为什么我应该尽力搞懂 monads？

互联网上充斥着如此之多的 monad 教程，足以表明理解这一概念是多难的一件事情。这是由于 monad 非常的抽象，同时它们又被用于各种不同的场景中，这可能会使人们难以真正的理解 monad 到底是什么以及它到底有什么用。

在Haskell中，monad 在 I/O 系统中起着核心作用。在 Haskell 中执行 I/O 的并不需要掌握 monad，但是了解 I/O monad 将会改进您的代码并扩展您的能力。

对于程序员而言，monad 是用于构造函数式程序的有用工具。它们具有三个使它们特别有用的属性：

1. 模块化 - 它们允许由更简单的计算组成计算，并将 计算的组合 与 计算执行 分开。

2. 灵活性 - 与没有 monad 编写的等效程序相比，它们使函数式程序具有更大的适应性。这是因为 monad 将计算策略提炼到一个地方，而不是将它们分散到整个程序的各处。

3. 隔离 - 它们可用于创建命令式风格的计算结构，这些结构与函数式程序的主体保持安全隔离。这对于将副作用（例如 I/O ）和状态（违反引用透明性）合并到像 Haskell 这样的纯函数式语言中很有用。

上面的每一个特性都将在本教程的后面特定 monad 的上下文中进行重新介绍。
