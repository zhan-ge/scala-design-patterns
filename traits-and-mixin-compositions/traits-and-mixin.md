# 特质与混入组合

在专研一些实际的设计模式之前，我们需要确保读者对 Scala 中的一些概念都很了解。这些概念中的很多一部分都会在后续用于实现实际的设计模式，同时能够意识到这些概念的能力、限制以及陷阱也是编写正确、高效代码的关键因素。尽管有些概念并不被认为是“官方上”的设计模式，但仍然可以使用它们来编写好的软件。甚至有些场景中，基于 Scala 的丰富特性，一些概念甚至可以使用语言特性来代替一种设计模式。最后，向我们前面提到的，设计模式的存在是因为一种语言的特性缺失，因为特性的不够丰富而不能完成一些需要的任务。

在第一个主题中我们将会了解**特质**与**混入(mixin)**组合。他们为开发者提供了一种可能来共享已实现的功能或在应用中为类定义接口。很多由特质与混入组合为开发者带来的可能性对于本书后续要实现的一些设计模式都是很有帮助的。本章将主要关注一下几个主题：

- 特质
- 混入组合
- 多重继承
- 线性化
- 测试特质
- 特质与类

