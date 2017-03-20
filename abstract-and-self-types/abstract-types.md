# 抽象类型

参数化一个类的常用方式之一是使用值。这十分简单，可以通过向类的构造器参数传入不同的值来实现。在下面的例子中，我们可以向`Person`类的`name`参数传入不同的值来创建不同的实例：

```scala
case class Person(name:String)
```

以这种方式我们能够创建不同的实例并将他们加以区分。但这样既不有趣也不是“火箭科学”。更进一步，我们会关注一些更加有趣的参数化来帮助我们编写更好的代码。

## 泛型

泛型是另一种参数化类的方式。当我们编写一个要操作多种不同类型的功能时，泛型会很有用，同时我们能够简单的推迟到最后再选择具体类型。一个例子是开发者所熟知的集合类。比如`List`，可以保存任何类型的数据，我们可以拥有整形、双精度浮点型、字符串、自定义类等等类型的列表。即便如此，列表的实现仍然总会是一样的。

我们同样可以参数化方法。比如，如果我们想要实现加法，对于不同的数字类型不会有什么不同之处。因此，我们可以使用泛型并仅实现方法一次，而不需要重载来适应世界上的每个类型。

让我们看一些例子：

```scala
trait Adder{
  def sum[T](a: T, b: T)(implicit numeric:Numeric[T]): T = 
    numeric.plus(a, b)
}
```

上面这段代码可能有点难懂，它定义了一个可以用于*numeric*类型的`sum`方法。这实质上是一个专用(ad-hoc)泛型的表示，我们将会在本章的后续部分讨论。

下面的代码展示了如何泛型一个能够包含任何数据的类：

```scala
class Container[T](data: T) {
  def compare(other:T) = data.equals(other)
}
```

下面的片段展示了例子的用法：

```scala
object GenericsExample extends App with Adder {
  println(s"1 + 3 = ${sum(1, 3)}")
  println(s"1.2 + 6.7 = ${sum(1.2, 6.7)}")
  
  // compilation fails
  // System.out.println(s"abc + cde = ${sum("abc", "cde")}") 
  
  val intContainer = new Container(10)
  println(s"Comparing with int: ${intContainer.compare(11)}")
  
  val stringContainer = new Container("some text")
  println(s"Comparing with string: ${stringContainer.compare("some text")}")
}
```

运行程序将会得到如下输出：

```scala
1 + 3 = 4
1.2 + 6.7 = 7.9
Comparing with int: false
Comparing with string: true
```

## 抽象类型

另一种参数化类的方式是使用抽象类型。泛型在其他语言中都有对应的实现，比如 Java。但是 Java 中却不存在抽象类型。让我们看一下上面`Container`的例子如何转换为以抽象类型的方式实现：

```scala
trait ContainerAT{
  type T
  val data:T
  
  def conpare(other:T) = data.equals(other)
}
```

然后在类中使用这个特质：

```scala
class StringContainer(val data:String) extends ContainerAT {
  override type T = String
}
```

然后就可以使用与前面相同的方式来使用这个类：

```scala
object AbstractTypesExample extends App{
  val stringContainer = new StringContainer("some text")
  println(s"Comparing with string: ${stringContainer.compare("some text")}")
}
```

同样可以得到与预期一样的输出：

```scala
comparing with string: true
```

当然我们也可以以类似的方式应用于泛型的例子，只需要创建一个特质的实例然后指定类型参数。这意味着泛型和抽象类型为我们提供了两种方式来实现相同的一件事。

## 泛型与抽象类型

那为什么 Scala 中同时拥有泛型和抽象类型呢？它们有什么不同吗？或者如何选择使用哪一个呢？我们会在这里给你答案。

泛型和抽象类型是可以互换的。虽然可能需要额外的工作，但是我们能够使用泛型来提供抽象类型所带来的一切。如何选择取决于不同的因素，有的是个人偏好，比如有人是为了可读性，而有人则是为了类的不同用法。

让我们看一个例子来尝试理解泛型与抽象类型可以在合适以及如何使用。在这个例子中我们会使用打印机。大家都知道它们有多种类型——纸质打印机、3D 打印机等等。每种都是用不同的材料来打印，比如墨粉、墨水或塑料，同时它们也会于打印到不同的媒介上——纸或甚至是空气中。我们可以使用抽象类型来描述这些：

```scala
abstract class PrintData
abstract class PrintMaterial
abstract class PrintMedia
trait Printer{
  type Data <: PrintData
  type Material <: PrintMaterial
  type Media <: PrintMedia
  
  def print(data:Data, material:Material, media: Media) = 
    s"Printing $data with $material material on $media media."
}
```

为了能够调用这个`print`方法，我们需要拥有不同的媒介、数据类型及原料：

```scala
case class Paper() extends PrintMedia
case class Air() extends PrintMedia
case class Text() extends PrintData
case class Model() extends PrintData
case class Toner() extends PrintMaterial
case class Plastic() extends PrintMaterial
```

现在让我们创建两个具体的打印机实现，一个激光打印机，一个 3D 打印机：

```scala
class LaserPrinter extends Printer {
  type Media = Paper
  type Data = Text
  type Material = Toner
}

class ThreeDPrinter extends Printer {
  type Media = Air
  type Data = Model
  type Material = Plastic
}
```

在上面的代码中，我们实际上已经给出了数据类型、媒介，以及打印机可以使用的材料的说明。我们不能要求 3D 打印机使用墨粉来打印，或者激光打印机直接打印在空气中。下面是如何使用这两个打印机：

```scala
object PrinterExample extends App{
  val laser = new LaserPrinter
  val threeD = new ThreeDPrinter
  
  println(laser.print(Text(), Toner(), Paper()))
  println(threeD.print(Model(), Plastic(), Air()))
}
```

这段代码拥有很好的可读性，它支持我们轻松的指定具体类。使事物更易于建模。有意思的是将其转换为泛型的方式实现则会是这样：

```scala
trait GenericPrinter[Data <:PrintData, Material <: PrintMaterial, Media <: PrintMedia] {
  def print(data: Data, material: Material, media: Media) =
    s"Printing $data with $material material on $media media."
}
```

这个特质很容易被描述，可读性和逻辑性在这里也没有得到损害。然而，我们必须以如下方式来表示具体类：

```scala
class GenericLaserPrinter[Data <: Text, Material <: Toner, Media <: Paper] extends GenericPrinter[Data, Material, Media]

class GenericThreeDPrinter[Data <: Model, Material <: Plastic, Media <: Air] extends GenericPrinter[Data, Material, Media]
```

这会让具体类的定义变得相当长，开发者也更可能犯错。下面的片段展示了如何使用这些类来创建实例：

```scala
val genericLaser = new GenericLaserPrinter[Text, Toner, Paper]
val genericThreeD = new GenericThreeDPrinter[Model, Plastic, Air]
println(genericLaser.print(Text(), Toner(), Paper()))
println(genericThreeD.print(Model(), Plastic(), Air()))
```

你会发现每次在创建这些实例的时候都需要指定类型。假如我们拥有更多的泛型类型，或者一些类型本身又是基于泛型，比如集合。这很快会变得冗长，而且让人难以理解这些代码的实际用途。

另一方面，使用泛型能够允许我们复用`GenericPrinter`而不必为不同的打印机表示进行显式的子类化。然而这也存在逻辑错误的风险：

```scala
class GenericPrinterImpl[Data <: PrintData, Material <: PrintMaterial, Media <: PrintMedia] extends GenericPrinter[Data, Material, Media]
```

如果像相面这样使用则会有犯错的危险：

```scala
val wrongPrinter = new GenericPrinterImpl[Model, Toner, Air]
println(wrongPrinter.print(Model(), Toner(), Air()))
```

## 使用建议

上面的例子展示了使用泛型和抽象类型的简单比较。两种都是有用的概念；然而，清晰的知道他们具体的用法对于在需要的场景选择正确的一个是很重要的。下面的一些技巧能够帮助你做出正确的选择：

- 泛型：
  - 如果你仅需要类型实例化。一个好的示范是标准的集合类。
  - 如果你正在创建一族类型。
- 抽象类：
  - 如果你想允许别人能够通过特质混入类型。
  - 如果你需要在一些允许互换的场景拥有更好的可读性。
  - 如果你想在客户端代码中隐藏类型定义。