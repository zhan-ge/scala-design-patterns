# 代数数据类型和类层级

代数数据类型和类层级是 Scala 中的另一种统一化。在其他的函数式语言中都拥有不同的方式来创建自定义的代数数据类型。在 Scala 中，这是通过类层级来实现的，称为`case class`和`object`。让我们看一下 ADT 到底是什么，他们是什么类型，以及如何定义它们。

## ADTs

代数数据类型只是一种组合类型，它会组合一些已存在的类型或仅表示一些新类型。它们中仅包含数据而非像常规类一样包含任何数据之上的功能。比如，一周之中的一天，或者一个表示 RGB 颜色的类——他们没有任何功能，仅仅是包含类一些信息。下面的几个小节将会带你洞悉 ADT 是什么以及它们是什么类型。

### 概括 ADT

整体 ADT 是一种我们可以简单的枚举一个类型所有可能的值并为每个值提供一个单独的构造器。作为一个例子，我们考虑一下一年的所有月份。一年中仅有 12 个月，而且不会改变：

```scala
sealed abstract trait Month
case object January extends Month
case object February extends Month 
case object March extends Month 
case object April extends Month 
case object May extends Month 
case object June extends Month 
case object July extends Month 
case object August extends Month 
case object September extends Month 
case object October extends Month 
case object November extends Month 
case object December extends Month

object MonthDemo extends App {
  val month:Month = February
  println(s"The current month is: $month")
}
```

运行这个程序将会得到如下输出：

```bash
The current month is: February
```

> 上面代码中的`Month`是密闭的(使用 sealed 关键字声明)，因为我们不想在当前文件之外被扩展。你可以发现，我们将不同的月份定义为对象，因为没有动机让他们成为分开的实例。值就是他们本身，同时也不会改变。

### 产品 ADT

在产品 ADT 类型中，我们无法枚举所有可能的值。手写他们通常会非常多。我们不能为每个单独的值提供各自的构造器。

比如颜色，有不同的颜色模型，不过最常见的是 RGB。它将主要颜色的不同值进行组合(红绿蓝)以表示其他颜色。假如每种颜色的值区间为 0 - 255，这意味着要表示所有可能的颜色，我们需要 256 的三次方个构造器。因此我们可以使用一个产品 ADT：

```scala
sealed case class REB(red: Int, green:Int, blue:Int)

object RGBDemo extends App {
  val magenta = RGB(255, 0, 255)
  println(s"Magenta in RGB is: $magenta")
}
```

> 可以发现，对于产品 ADT 来说，所有不同的值拥有同一个构造器。

### 混合 ADT

混合 ADT 表示一个概况和产品的组合。这意味我们可以拥有特定的值构造器，不过这些值构造器同时接收参数以包装其他类型。

让我们看一个例子。加入我们正在编写一个绘图程序：

```scala
sealed abstract trait Shape
case class Circle(radius: Double) extends Shape
case class Rectangle(height:Double, width:Double) extends Shape
```

我们拥有不同的形状。这个例子中展示了一个概况 ADT，因为我们拥有`Circle`和`Rectangle`两个特定的值构造器。同时我们也拥有一个产品 ADT，因为这两个特定的值构造器同时接收额外的参数以表示更多的类型。

我们把这个类展开一点。当绘制形状时，我们需要知道其位置。因此我们可以添加一个 Point 类来持有 x 和 y 坐标：

```scala
case class Point(x:Double, y:Double)

sealed abstract trait Shape
case class Circle(centre:Point, radius:Double) extends Shape
case class Rectangle(topLeft:Point, height:Double, width:Double) extends Shape
```

希望这有助于理解 Scala 中的 ADT 是什么以及如何使用它们。

### 统一化

在所有上面的例子之后，可以明显的发现类层级和 ADT 是被统一化的而且看起来是同一件事。这给语言添加了一个更高级别的灵活性，使相其对于其他函数式语言中的建模则更加容易。

## 模式匹配

模式匹配常用于 ADT。当基于 ADT 的值来做什么事的时候它会使代码更清晰易读，相对于`if-else`语句也更易扩展。你可能会认为这些语句在有些场景会难以处理，特别是某个数据类型拥有多种可能的值时。

### 对值进行模式匹配

在之前月份的例子中，我们仅拥有月份的名字。我们或许需要他们的数字，否则电脑将不知道这是什么。下面展示了做法：

```scala
object Month {
  def toInt(month: Month): Int = month match { 
    case January => 1 
    case February => 2 
    case March => 3 
    case April => 4 
    case May => 5 
    case June => 6 
    case July => 7 
    case August => 8 
    case September => 9 
    case October => 10 
    case November => 11 
    case December => 12
  }
}
```

你可以看到我们基于他们来匹配不同的值，并最终返回这行正确的值。现在可以这样使用该方法：

```scala
println(s"The current month is: $month and it's number ${Month.toInt(month)}")
```

和预期一样，我们的应用会生成如下输出：

```scala
The current month is: February and it's number 2
```

实际上因为我们指定了基特质为密闭约束，因此在我们的代码之外没人能够扩展它们，同时我们拥有一个没有遗漏的模式匹配。不完整的模式匹配将会充满问题。作为实验，如果我们将二月份注释掉并编译我们的代码，将会得到如下警告：

```scala
Warning:(19, 5) match may not be exhaustive.
It would fail on the following input: February
	month match {
	^
```

运行这个例子将会证明这个警告的正确性，当传入参数使用`February`时我们的代码将会失败。为了完整，我们可以添加一个默认的情况：

```scala
case _ => 0
```

### 对产品类型进行模式匹配

当模式匹配用于产品或混合 ADT 时将会展示其强大力量。在这种情况下，我们可以匹配这些数据类型的实际值。让我们展示如何实现一个功能来计算面积的值：

```scala
object Shape{
  def area(shape: Shape):Double = shape match {
    case Circle(Point(x, y), radius) => Math.PI * Match.pow(radius, 2)
    case Rectangle(_, h, w) => h * w
  }
}
```

在匹配时可以忽略我们不关心的值。对于面积，我们实际上不需要位置信息。在上面的代码中，我们仅仅展示了匹配的两种方式。下划线`_`操作符可以用于`match`语句的任何位置，它仅仅会忽略所处位置的值。在这之后，直接使用我们的例子：

```scala
object ShapeDemo extends App {
  val circle = Circle(Point(1, 2), 2.5)
  val rect = Retangle(Point(6, 7), 5, 6)
  
  println(s"The circle area is: ${Shape.area(circle)}")
  println(s"The rectangle area is: ${Shape.area(rect)}")
}
```

可以得到类似下面的输出：

```scala
The circle area is: 19.634954084936208 
The rectangle area is: 30.0
```

在模式匹配时，我们甚至可以使用常量来代替变量来作为 ADT 的构造器参数。这使语言更加强大并支持我们实现更加复杂的逻辑，并且看起来也会很不错。你可以根据上面的例子进行试验以便更深入理解模式匹配的工作原理。

> 模式匹配常用语 ADT，它有助于实现清晰、可扩展及无遗漏的代码。

