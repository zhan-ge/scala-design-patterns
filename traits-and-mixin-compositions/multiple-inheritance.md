---
typora-copy-images-to: ../traits-and-mixin-compositions
---

# 多重继承

因为可以同时混入多个特质，而且这些特质都可以拥有各自不同的方法实现，因此我们已经在前面的章节中多次提到了多重继承。多重继承不仅是一个强大的技术，同时也很危险，甚至有些语言中决定不进行支持，比如 Java。向我们看到的一样，Scacla 对此进行了支持，不过带有一些限制。本节中我们会接收多重继承的问题及 Scala 是如何处理这些问题的。

## 菱形问题

多重继承忍受着菱形问题的痛苦。

让我们看一下下面的图示：

![5623E962-D09D-42FD-BC31-F26511ABF96C](/assets/diamond.png)

如图，B 和 C 同时扩展了 A，然后 D 同时扩展了 B 和 C。这看起来可能不是很清晰。比如我们有个方法一开始定义在 A，但是 B 和 C 同时覆写了它。当在 D 中调用该方法时会发生什么呢？实际上调用的是哪个方法呢？

因为上面的问题有点模糊或将引起错误。让我们使用 Scala 的特质来重新描述一下该问题：

```scala
trait A {
  def hello(): String = "Hello from A"
}

trait B extends A {
  override def hello(): String = "Hello from B"
}

trait C extends A {
  override def hello(): String = "Hello from C"
}

trait D extends B with C {
  
}

object Diamond extends App with D {
  println(hello())
}
```

运行后会得到如下结果：

```bash
Hello from C
```

如果我们把特质 D 修改为：

```scala
trait D extends C with B {

}
```

则会得到结果为：

```bash
Hello from B
```

你会发现，尽管例子仍然会有点模糊甚至易于出错，我们可以告诉你实际上被调用的是哪个方法。这是通过**线性化(linearization)**实现的，在下一节中会深入介绍。

## 限制

在我们关注线性化之前，让我们指出 Scala 所支持的多重继承拥有的限制。我们之前已经见过他们很多次，这里会概括描述。

> **Scala 多重继承限制**
>
> Scala 中的多重继承由特质实现并遵循线性化规则。
>
> 在多重继承中，如果一个特质已经显式扩展了一个类，则混入该特质的类必须是之前特质混入的类的子类。这意味着当混入一个已扩展了别的类的特质时，他们必须拥有相同的父类。
>
> 如果特质中**定义**或**声明**了相同签名但返回类型不同的方法，则无法混入这些特质。

需要特别小心的是特质中**定义**了相同签名和返回类型的方法。若果方式仅是**声明**而被要求实现，这不会带来问题而且只需要提供一个实现即可。

