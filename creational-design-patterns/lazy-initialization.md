# 懒初始化

在软件工程中，懒初始化是是将对象或变量的初始化过程延迟直到第一次被需要使用的时候。这个想法的背后是想要延迟或避免昂贵操作的执行。

## 类图

在其他的语言中，比如 Java，懒初始化通常与工厂方法模式联合使用。该方法通常会检查我们需要使用的对象或变量是否已经被初始化了，如果没有则初始化该对象并返回。在后续的使用中，将会直接返回已被初始化的对象或变量。

Scala 语言中对懒初始化提供了内建支持，通过`lazy`关键字。因此在这里提供类图也没有意义了。

## 实例

让我们看一下懒初始化在 Scala 中是如何工作的，并证明它真的是“懒”的。我们会看一个求圆形面积的计算。我们知道，公式是`pi * r² ` 。数学语言中已经提供了数学常量，我们在现实生活中也不会这么做。然而，如果我们讨论的是一个不为大家熟知常量，这个例子仍然是有关联的，或者基于一个值波动的常量，但是每天都会不同。

在学习里我们被教育 π 的值为 3.14。这是对的，然而，如果我们关心精度的话后面还有很多额外的小数，我们仍然要包括它们。比如，包含 100 个小数的 π：

```
3.14159265358979323846264338327950288419716939937510582097494459230781 64062862089986280348253421170679
```

因此让我们创建一个根据半径计算圆形面积的工具。我们将会把我们的 π 值作为变量放在工具类中，不过我们仍然支持用户来选择是否需要一个精确的面积值。如果需要，我们将会从配置文件中读取一个包含 100 个小时的 π 值：

```scala
object CircleUtils{
  val basicPi = 3.14
  lazy val precisePi:Double = {
    println("Reading properties for the precise PI.")
    val props = new Properties()
    props.load(getClass.getResourceAsStream("pi.properties"))
    props.getProperty("pi.high").toDouble
  }
  
  def area(redius:Double, isPrecise:Boolean = false):Double = {
    val pi:Double = if (isPrecise) precisePi else basicPi
    pi * Math.pow(radius, 2)
  }
}
```

基于对精度的需求，我们将会使用 π 的不同版本。这里的懒初始化是很有帮助的，因为我们可能永远都不需要计算一个精确的面积。或者有时需要有时不需要。更进一步，读取配置文件是一个 IO 操作，可以认为是一个慢的或者多次执行会产生不好的副作用的操作。现在来使用它：

```scala
object Example extends App {
  System.out.println(s"The basic area for a circle with radius 2.5 is ${CircleUtils.area(2.5)}")
  System.out.println(s"The precise area for a circle with radius 2.5 is ${CircleUtils.area(2.5, true)}")
  System.out.println(s"The basic area for a circle with radius 6.78 is ${CircleUtils.area(6.78)}") 
  System.out.println(s"The precise area for a circle with radius 6.78 is ${CircleUtils.area(6.78, true)}")
}
```

我们可以运行并观察程序的输出。首先，精度确实很重要，比如一些行业，金融、航天等等要重要的多。然后，在懒初始化块中，我们使用了一个`print`方法，它会在我们第一次使用精确实现的时候打印信息。普通的值会在实例创建的时候完成初始化。这显示了 Scala 中的懒初始化确实在第一次使用的时候才执行。

## 优点

当一个对象或变量的初始化过程很耗时或者可能用不着，懒初始化会尤其有用。有人会说我们可以简单的使用一个方法来实现，这也是对的。然而，假如一种场景，我们需要在我们的对象中以不同的调用从多个方法访问一个懒初始化的对象或变量。这种情况下，将结果保存在一个地方并复用它将会很有帮助。

## 缺点

在 Scala 之外的语言中，从多线程环境访问一个懒初始化的变量需要被小心管理。在 Java 中，需要使用`synchronized`来进行初始化。为了能够提供更好的安全性，应该优先使用*double-checked locking*，而 Scala 中则没有这种危险(?)。

