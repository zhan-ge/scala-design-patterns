# 多态

多态是任何使用面向对象编程语言的开发者都知道的东西。

> 多态帮助我们编写通用的代码以进行复用并应用到多样性的类型中。

知道有多种不同类型的多态是很重要的，这节中我们将会讨论它们。

## 子类型多态

这是一种每个开发者都知道的多态，它与在具体类中覆写方法相关。考虑下面简单的层级：

```scala
abstract class Item {
  def pack:String
}

class Fruit extends Item {
  override def pack:String = "I'm a fruit and I'm packed in a bag."
}

class Drink extends Item {
  override def pack: String = "I'm a drink and I'm packed in a bottle."
}
```

现在我们拥有一个装满物品的购物篮，对每一个都进行`pack`调用：

```scala
object SubtypePolymorphismExample extends App {
  val shoppingBasket:List[Item] = List(
    new Fruit, new Drink
  )
  shoppingBasket.foreach(i => println(i.pack))
}
```

你会发现，这里我们可以使用一个抽象类型并且无需思考它们具体的类型直接调用`pack`方法即可。多态会注意打印正确的值。我们的输出会像下面这样：

```
I'm a fruit and I'm packed in a bag. 
I'm a drink and I'm packed in a bottle.
```

> 子类化多提通过`extends`关键字使用继承来表示。

## 参数式多态

函数式编程中的参数化多态即为我们上节中展示的泛型。泛型既是参数化多态，我们已经见过，它支持我们定义基于任何类型或给定类型的子集类型的方法和数据结构。而具体的类型则可以在后续的阶段指定。

> 参数式多态使用类型参数表示。

## 专用(ad-hoc)多态

专用多态类似于参数式多态；然而在这种情况下，参数的类型是很重要的，因为具体实现会依赖于这些参数。它会在编译时被分析，不像子类型多态是在运行时。这多少类似于函数重载。

我们在本章的前面部分看到过它的一个例子，我们创建了一个`Adder`特质可以对不同的类型求和。让我们一步一步定义一个更加精密的例子，期望这样有助于连接它是如何工作的。我们的目标是让`sum`方法可以对任何类别的类型(之前是数字类型)求和：

```scala
trait Adder[T]{
  def sum(a:T, b:T): T
}
```

下一步，我们创建一个使用`sum`方法的对象并向外界暴露：

```scala
object Adder {
  def sum[T: Adder](a: T, b: T): T = 
    implicitly[Adder[T]].sum(a, b)
}
```

我们看到的上面这段代码是 Scala 中的一些语法糖，`implicitly`表示拥有一个隐式转换可以将`T`转换为`Adder[T]`。现在我们可以编写如下程序：

```scala
object AdhocPolymorphismExample extends App{
  import Adder._
  println(s"The sum of 1 + 2 is ${sum(1, 2)}")
  println(s"The sum of abc + def is ${sum("abc", "def")}")
}
```

如果我们尝试编译运行这个程序，将会得到如下错误：

```
Error:(15, 51) could not find implicit value for evidence parameter 
of type com.ivan.nikolov.polymorphism.Adder[Int]
	System.out.println(s"The sum of 1 + 2 is ${sum(1, 2)}")
												  ^
Error:(16, 55) could not find implicit value for evidence parameter 
of type com.ivan.nikolov.polymorphism.Adder[String]
	System.out.println(s"The sum of abc + def is ${sum("abc", "def")}")
													  ^
```

这表示我们的代码不知道如何将整数或字符串隐式转换为`Adder[Int]`或`Adder[String]`。我们需要做的是定义这些转换以告诉程序`sum`方法该如何做。我们的`Adder`对象看起来会是这样：

```scala
object Adder {
  def sum[T: Adder](a: T, b: T): T = implicitly[Adder[T]].sum(a, b)
  
  implicit val int2Adder:Adder[Int] = new Adder[Int] {
    override def sum(a:Int, b:Int):Int = a + b
  }
  
  implicit val string2Adder:Adder[String] = new Adder[String] {
    override def sum(a:String, b:String):Int = s"$a concatenated with $b"
  }
}
```

现在如果允许上面的程序则会得到如下输出：

```
The sum of 1 + 2 is 3
The sum of abc + def is abc concatenated with def
```

同样，如果你记得本章开头的例子，我们是不能对字符串求和的。现在你会发现，我们可以提供不同的实现，因为我们定义了一个转换为`Adder`方式，因此使用任何类型都是没有问题的。

专用泛型支持我们在不修改基类的基础上扩展代码。这在使用外部库时将会很有帮助，比如因为某些原因我们无法修改原始代码。它很强大而且在编译时求解，这可以确保我们的程序会和预期的一样运行。另外，它可以支持我们为无法访问的类型(比如这里的 Int、String)提供功能。

### 为多个类型提供功能

如果我们回头看本章的开头，我们为数字类型定义了一个`Adder`，会发现后面最终的实现会要求我们为不同的数字类型单独定义不同的操作，比如 Long、Double等等。有没有方式来实现本章开头的哪种效果呢？当然有，就像下面这样：

```scala
implicit def numeric2Adder[T: Numeric]:Adder[T] = new Adder[T]{
  override def sum(a: T, b: T) = implicitly[Numeric[T]].plus(a, b)
}
```

我们仅仅定义了另一个隐式转换(将`Numeric[T]`转换为`Adder[T]`)，它会为我们处理好一切。现在可以像下面这样使用：

```scala
println(s"The sum of 1.2 + 6.5 is ${sum(1.2, 6.5)}")
```

> 专用多态通过**隐式混入行为**来表示。这是**类型类设计模式**的主要构建方式，后续章节中将会详细解释。
>
> 例子中，首先定义一个泛型接口，然后通过传入具体类型参数来创建不同类型的具体子类实例，从而调用支持不同类型的实例方法。只是这个过程是通过隐式转换完成的。

