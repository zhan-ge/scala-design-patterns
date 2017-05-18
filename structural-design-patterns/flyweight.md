---
typora-copy-images-to: ../assets
---

享元模式

通常当软件编写完成后，开发者会尝试使其更加快速高效。通常这意味着更少的处理循环和更少的内存占用。有多种不同的方式来实现这两个概念。大多数时间，一个好的算法能够处理好第一个问题。内存的使用规模则存在不同的原因和解决方案，而享元模式则是用于帮助减少内存的使用。

> 该模式的目是通过将一个对象与其类似的对象共享尽可能多的数据来帮助优化内存的使用。

在很多情况下很多对象会共享一些相同的信息。讨论享元模式时一个常用的例子是单词处理。替代使用所有的信息包括字体、大小、颜色、图片等等来表示一个字符，我们可以仅仅保存类似字符的坐标并引用一个拥有相同信息的对象。这样可以使内存的使用显著减少。否则，这样的应用将无法使用。

类图

对于类图，让我们假设正在尝试表示一个类似下面这样用于色盲测试的图片：

![color-bindless](/assets/color-bindless.png)

我们可以发现，它由拥有不同大小和颜色的原型组合而成。可能的情况下，这可能是一个无限大的图片从而拥有无数个原型。为了使事情变的简单，让我们设置一个限制，假设仅拥有五种不同的颜色：红、绿、蓝、黄、洋红。下面的类图展示了如何使用享元模式来表示类似上面的图片：

![CEF6718B-A982-4FC9-B802-6115EAFC355D](/assets/flyweight.png)

实际的享元模式通过`CircleFactory`、`Circle`、`Client`来实现。客户端请求工厂，要么返回一个新的`Circle`实例，要么是拥有相同参数的实例已经存在，则会从缓存中返回。比如，共享的数据是带有各自颜色的`Circle`对象，而每个特定的`Circle`都拥有各自的坐标和半径。`Graphic`中会包含带有所有信息的圆形。基于上面类图的代码实现会使一切变得更加清晰。

实例

是时候看一下使用 Scala 来表示享元模式是什么样子了。我们将使用前面展示过的相同的例子。如果代码版本中的名称等属性与类图中有所不同，并没有什么值得注意的。这种变化主要是源自 Scala 的习俗。在贯穿代码的时候我们将会在发生的地方明确指出。

关于享元模式和例子中有趣的一点是我们实际上使用了一些前面提到过的模式和技术。我们同样会在贯穿代码的时候明确指出。

首先我们要做的是表示颜色。这实际上跟模式无关，这里我们使用 ADT 来表示：

```scala
sealed abstract class Color
case object Red extends Color
case object Blue extends Color
case object Green extends Color
case object Yellow extends Color
case object Magenta extends Color
```

在定义完颜色之后，我们开始定义`Circle`类：

```scala
class Circle(color:Color){
  System.out.println(s"Creating a circle with $color color.")
  
  override def toString(): String = s"Circle($color)"
}
```

这些圆形将会是享元对象，因此对象中仅拥有将会与其他对象共享的数据。现在我们拥有了圆形的模型，现在创建圆形工厂。这里使用了工厂模式：

```scala
object Circle{
  val cache = Map.empty[Color, Circle]
  
  def apply(color: Color): Circle = 
    cache.getOrElseUpdate(color, new Circle(color))
    
  def circlesCreated(): Int = cache.size
}
```

我们使用 Scala 中的伴生对象来实现工厂模式。这也就是为什么类名与上面类图中的类名不同。这种表示支持我们使用下面的语法要么获得一个已存在的实例要么创建一个新的实例：

```scala
Circle(Green)
```

现在我们拥有了圆形和工厂，然后实现`Graphic`类：

```scala
class Graphic{
  val items = ListBuffer.empty[(Int,Int,Double,Circle)]
  
  def addCircle(x:Int, y:Int, radius:Double, circle:Circle):Unit = {
    items += ((x,y,radius,circle))
  }
  
  def draw():Unit = {
    items.foreach {
      case (x, y, radius, circle) =>
        System.out.println(s"Drawing a circle at ($x, $y) with radius $radius: $circle")
    }
  }
}
```

`Graphic`类实际上将会包含所有其他与圆形相关的数据。上面类图中的`Client`并不需要在代码中进行明确的表示——它仅仅是使用工厂的代码。类似的，`Graphic`对象会被程序来搜索圆形对象，而非通过一个客户端的显式访问。下面是对这些类的使用：

```scala
object FlyweightExample extends App{
  val graphic = new Graphic
  graphic.addCircle(1, 1, 1.0, Circle(Green)) 
  graphic.addCircle(1, 2, 1.0, Circle(Red)) 
  graphic.addCircle(2, 1, 1.0, Circle(Blue)) 
  graphic.addCircle(2, 2, 1.0, Circle(Green)) 
  graphic.addCircle(2, 3, 1.0, Circle(Yellow)) 
  graphic.addCircle(3, 2, 1.0, Circle(Magenta)) 
  graphic.addCircle(3, 3, 1.0, Circle(Blue)) 
  graphic.addCircle(4, 3, 1.0, Circle(Blue)) 
  graphic.addCircle(3, 4, 1.0, Circle(Yellow)) 
  graphic.addCircle(4, 4, 1.0, Circle(Red))

  graphic.draw()

  System.out.println(s"Total number of circle objects created: ${Circle.circlesCreated()}")
}
```

在一开始定义`Circle`类的时候，我们会在其构造期间打印一条信息。运行这段代码会发现，带有每个特定颜色的圆形仅会被创建一次，尽管我们在构件整个图的时候使用了很多圆形。最后一行信息显示了我们事实上仅拥有 5 个实际的圆形实例，尽管我们的图拥有十个不同的圆形。

该实例仅仅展示了享元模式是如何工作的。在真实的应用中，享元对象可能会共享更多的信息，从能能够在实际的应用中减少内存的占用。

## 优点

像我们前面已经提到的，享元模式可以用于尝试减少应用的内存占用。使用共享的对象，我们的应用会需要更少对象的构造和解构，同时也能提高性能。

## 缺点

基于所要共享数据的规模，有时实际共享对象的数量会动态的增长而不能带来更多的优势。另外，会增加工长本身和其用法的复杂性。多线程应用则需要对工厂进行额外的处理。最后但并非最不重要的是，开发者在使用共享对象时需要额外小心，其中的任何改变都会影响到整个应用，在 Scala 中由于不可变性则不需要额外关心。

