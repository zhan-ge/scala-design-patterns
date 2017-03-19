# 线性化

我们已经看到，特质提供了一种多重继承的形式。在这行情况下，继承并非必须是线性的，但是形成了一个需要在编译中展开的无环图。而线性化所做的就是：它为一个类的所有父级指定一个线性顺序，包括常规的超类链和所有其他特质的父链。

对于那些不包含代码的特质并不需要进行线性化。然而，如果使用了混入，我们则需要为此考虑。如下两项将会被线性化影响：

- 方法定义
- 变量(var/val)

我们子啊前面已经看到过一个线性化的简单实例。然而，如果线性化规则不清晰的话，事情会变得非常复杂且无法预料。

## 继承层次规则

在深入线性化规则之前，我们需要清楚了解 Scala 中的继承规则：

- 在 Java 中，即便一个类没有显式扩展任何其他的类，它也会拥有一个`java.lang.Object`父级。Scala 也是这样，不过这个父级是`AnyRef`。
- 直接扩展一个特质和扩展特质的超类，以及混入特质都是相似的使用`with`关键字。

使用这些规则我们总能够得到一个所有特质和类的权威形式，基类使用`extends`指定，而其他所有特质使用`with`关键字。

## 线性化规则

Scala 中定义和存在的线性化规则是为了确保定义清晰的行为。规则的状态如下：

- 任何类的线性化必须包括它已扩展的任何**类**(不是特质)的未经修改的线性化。
- 任何类的线性化必须包括它已扩展的任何**特质**的 线性化中的 类和混入特质，但是混入的特质不会被限定于以他们混入的顺序出现在线性化中。
- 每个类或特质只能在线性化中出现一次。重复的则会被忽略。

我们已经在前面的例子中看到过，不能够混入拥有不同基类的特质，或让一个类混入与该类的基类不同的特质。

## 线性化是如何工作的

在 Scala 中，线性化会被从左到右列出，而更靠右的级别则会更高，比如`AnyRef`。在执行线性化的时候，`Any`同样会被添加到层级列表中。这会与线性化规则中的“任何类必须包括其超类的线性化”相结合，这意味着超类的线性化会以后缀的方式出现在类的线性化列表中。

让我们看一个实际的例子：

```scala
class Animal extends AnyRef
class Dog extends Animal
```

这两个类的线性化分别为：

```scala
Animal -> AnyRef -> Any
Dog -> Animal -> AnyRef -> Any
```

让我们尝试形式化一个算法来描述线性化是如何计算的：

1. 首先是一个类的声明：`class A extends B with T1 with T2`
2. 除了第一项，将该列表翻转，并去除关键字。这样 A 的超类 B 成了一个后缀：`A T2 T1 B`
3. 将每项都替换为其线性化：`A_ T2_ T1_ B_`
4. 以**右结合**级联操作的方式将列表连接：`A_ +: T2_ +: T1_ +: B_`
5. 追加标准的`AnyRef`和`Any`：`A_ +: T2_ +: T1_ +: B_ +: AnyRef +: Any`
6. 对上面的表达式求值。基于右结合级联操作，我们依次从右到左进行。在每一步中，我们将会移除掉已经出现在右手边的元素。在我们的例子中，当添加到`B_`时，将不会再添加已经包含的`AnyRef`和`Any`；仅添加一个`B_`并继续。在`T1_`的时候，我们将会跳过任何前面已经添加过的步骤等等，直到抵达 A。

最后，在线性化结束之后，我们将拥有一个没有重复的包含类和特质的列表。

## 初始化

现在我们知道线性化时发生了什么，我们将会理解实例是如何被创建的。规则是构造器代码将会以与线性化顺序相反的顺序执行。这意味着，从右到左，首先是`Any`和`AnyRef`的构造器被调用，然后是实际类的构造器。同样，超类的构造器或者任何混入的特质都会在实际类之前被调用，我们前面已经提到，他们会作为一个后缀被添加。

要知道从右到左贯穿线性化同时意味着在超类的构造器被调用之后，混入特质的构造器会被调用。它们会以他们出现在原有类定义中的顺序被调用(因为从右到左的方向，并且在一开始创建线性化的时候他们的顺序就被翻转了)。

## 方法覆写

当在子类中覆写方法时，你或许想要调用原有的实现。这可以通过在方法名之前添加`super`关键字作为前缀来实现。开发者同样可以通过`super`关键之来控制特质类型，以调用指定特质中的方法。我们已经在前面的章节中看到过这类用法，当时我们调用了`super[A].hello()`。在那个例子中我们混入了相同的方法；然而，方法本身并不会引用到超类(特质)中，但是仅仅定义了他们各自的实现。

让我们看一个例子，我们在覆写方法的同时引用了超类：

```scala
class MultiplierIdentity {
  def identity:Int = 1
}
```

现在定义两个特质分别将原有类中的标识乘以 2 倍和 3 倍：

```scala
trait DoubledMultiplierIdentity extends MultiplierIdentity {
  override def identity: Int = 2 * super.identity
}

trait DoubledMultiplierIdentity extends MultiplierIdentity {
  override def identity: Int =  3 * super.identity
}
```

和我们之前看到的例子一样，我们混入到特质的顺序造成了影响。我们会提供三种实现，首先是先混入`DoubledMultiplierIdentity`再混入`TripledMultiplierIdentity`。第一种实现中我们不会覆写方法，等同于使用`super`标示符：`super.identity`。其他两种实现则会覆写方法并引入指定的父级：

```scala
// first Doubled, then Tripled
class ModifiedIdentity1 extends DoubledMultiplierIdentity with TripledMultiplierIdentity

class ModifiedIdentity2 extends DoubledMultiplierIdentity with TripledMultiplierIdentity {
  override def identity: Int = super[DoubledMultiplierIdentity].identity 
}

class ModifiedIdentity3 extends DoubledMultiplierIdentity with TripledMultiplierIdentity {
  override def identity: Int = super[TripledMultiplierIdentity].identity 
} 
// first Doubled, then Tripled
```

让我们做和上面代码一样的事情，不过这次首先混入`TripledMultiplierIdentity`，然后是`DoubledMultiplierIdentity`：

```scala
// first Tripled, then Doubled 
class ModifiedIdentity4 extends TripledMultiplierIdentity with DoubledMultiplierIdentity

class ModifiedIdentity5 extends TripledMultiplierIdentity with DoubledMultiplierIdentity {
  override def identity: Int = super[DoubledMultiplierIdentity].identity 
}

class ModifiedIdentity6 extends TripledMultiplierIdentity with DoubledMultiplierIdentity {
  override def identity: Int = super[TripledMultiplierIdentity].identity 
} 
// first Tripled, then Doubled
```

最终使用这些类：

```scala
object ModifiedIdentityUser extends App{
  val instance1 = new ModifiedIdentity1 
  val instance2 = new ModifiedIdentity2 
  val instance3 = new ModifiedIdentity3 
  val instance4 = new ModifiedIdentity4 
  val instance5 = new ModifiedIdentity5 
  val instance6 = new ModifiedIdentity6

  println(s"Result 1: ${instance1.identity}") 
  println(s"Result 2: ${instance2.identity}") 
  println(s"Result 3: ${instance3.identity}") 
  println(s"Result 4: ${instance4.identity}") 
  println(s"Result 5: ${instance5.identity}")
  println(s"Result 6: ${instance6.identity}")
}
```

这个例子展示了一个多重继承层级结构，我们可以看到一个在前面解释其含义时提到的菱形关系。这里我们拥有了混入`DoubledMultiplier`和`TripledMultiplier`的所有可能的方式，和调用`identity`基类方法的方式。

因此这个程序将会得到什么样的输出的？一种可以预期的情况是当我们没有覆写方法时，它将会调用最右边的混入方法。因为两种情况下他们都会调用所扩展类的`super`方法，因此结果会是 2 和 3。让我们看一下：

```bash
Result 1: 6 
Result 2: 2 
Result 3: 6 
Result 4: 6 
Result 5: 6 
Result 6: 3
```

这个输出完全不符合预期。这也展示了 Scala 类型系统是如何工作的。在线性化场景中，当我们有一个多重继承时，对同一个方法的调用会基于特质在类定义中出现的顺序从右到左链接。注意，如果我们不使用`super`符号，则会打断这个链接，可以在上面的例子中发现这一点。

> 上面这个例子非常有趣，也证明了线性化是多么有必要知道以及线性化的原理。不清楚了解该特性将会导致一些列失误，从而导致在代码中引入决定性的错误。
>
> 我的建议仍然是尝试避免菱形继承的情况，尽管我也同意这种方式能够让复杂的系统的实现看起来更加简单及减少代码的编写量。然而一种情况就是像上面的例子一样让程序难以阅读并在未来难以维护。
>
> 你需要了解 Scala 中无处不在的线性化——而不仅仅是处理特质的时候。这也是 Scala 类型系统的工作方式。这意味着清楚了解构造器调用的顺序能够避免错误，也能让层级结构保持简单。





