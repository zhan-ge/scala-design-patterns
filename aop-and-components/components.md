# 组件

组件作为应用的一部分意味着会与应用的其他部分进行结合。它们应该是可复用的，以便减少代码的重复。组件通常拥有接口，用于描述它们提供的服务或者它们依赖的一些服务或是其他组件。

在大型的应用中，我们通常会看到多个组件会被集成在一起工作。要描述一个组件提供的服务通常会很直接，这会使用接口的帮助来完成。与其他组件进行集成则可能需要开发者完成更多的工作。这通常会通过**将需要的组件的接口作为参数来传递**。然而，加入有一个大型的应用需要很多的组件；完成这些链接需要花费时间和精力。进一步，每次需要一个新的需求，我们也需要进行大量的重构。**多重继承**可以作为参数传递的替代方案；然而，首先需要语言支持这种方式。

像 Java 语言中用来链接组件的流行做法是使用**依赖注入**。Java 中拥有这样的库用于在运行时将组件注入。

## 丰富的 Scala

本书中我们已经提到多次，Scala 比简单的面向对象语言拥有更强的表现力。我们已经讨论了一些概念，比如：抽象类型、自类型、统一化、混入组合。这支持我们创建通用的代码，特定的类，并能以相同的方式来处理对象、类、变量或函数，并实现多重继承。使用不同的组合用法可以让我们编写期望的模块化代码。

### 实现组件

作为一个例子，假如我们尝试构建一个做饭机器人。我们的机器人能够查找食谱并制作我们需要的菜肴。我们可以通过创建新的组件来给机器人添加新的功能。

我们期望代码是模块化的，因此有必要对功能进行拆分。下面的图示展示了机器人的雏形以及各组件间的关系：

![cook-robot](/assets/cook-robot.jpg)

首先让我们给不同的组件定义接口：

```scala
trait Time{
  def getTime():String
}

trait RecipeFinder{
  def findRecipe(dish:String):String
}

trait Cooker{
  def cook(what:String): Food
}
```

这个例子中需要一个简单的`Food`类：

```scala
case class Food(name:String)
```

一旦这些完成后，我们就可以开始创建组件了。首先是`TimeConponent`，而`Time`的实现是一个嵌套类：

```scala
trait TimeConponent{
  val time:Time
  
  class TimeImpl extends Time{
    val formatter = DateTimeFormatter.ofPattern("HH:mm:ss")
    override def getTime():String = 
      s"The time is: ${LocalDateTime.now.format(formatter)}"
  }
}
```

现在以类似的方式实现`RecipeComponent`，下面是组件和实现的代码：

```scala
trait RecipeComponent{
  val recipe:RecipeFinder
  
  class RecipeFinderImpl extends RecipeFinder{
    override def findRecipe(dish:String):String = dish match {
      case "chips" => "Fry the potatoes for 10 minutes."
      case "fish" => "Clean the fish and put in the oven for 30 minutes."
      case "sandwich" => "Put butter, ham and cheese on the bread, toast and add tomatoes."
      case _ => throw new RuntimeException(s"${dish} is unknown recipe.")
    }
  }
}
```

最终，我们需要实现`CookingComponent`。它实际上会使用`RecipeComponent`，下面是它的实现：

```scala
trait CookingComponent{
  this: RecipeComponent =>
  
  val cooker:Cooker
  
  class CookerImpl extends Cooker {
	override def cook(what:String):Food = {
      val recipeText = recipe.findRecipe(what)
      Food(s"We just cooked $what using the following recipe: '$recipeText'.")
	}
  }
}
```

现在所有的组件都各自实现了，我们可以将它们组合来创建我们的机器人。首先创建一个机器人要使用的组件注册表：

```scala
class RobotRegisty extends TimeComponent with ReipeComponent with CookingComponent {
  override val time:Time = new TimeImpl
  override val recipe:RecipeFinder = new RecipeFinderImpl
  override val cooker:Cooker = new CookerImpl
}
```

现在创建机器人：

```scala
class Robot extends RobotRegisty{
  def cook(what:String) = cooker.cook(what)
  def getTime() = time.getTime()
}
```

最后使用我们的机器人：

```scala
object RobotExample extends App {
  val robot = new Robot
  System.out.println(robot.getTime()) 
  System.out.println(robot.cook("chips")) 
  System.out.println(robot.cook("sandwich"))
}
```

上面的例子中，我们看到了 Scala 不使用外部库来实现依赖注入的方式。这种方式真的很有用，它会避免我们的构造器过大，也不需要扩展过多的类。更进一步，各个组件可以很好的分离，可测试，并能清晰定义各自的依赖。我们同样看到了可以使用一些依赖其他组件的组件来递归的添加需求。

> 上面这个例子实际上展示了蛋糕模式。一个好的特性是，依赖的存在会在编译期间进行求值，而不像 Java 那些流行的库一样在运行时进行求值。
>
> 蛋糕模式同样也存在缺点，我们会在稍后关注所有特性——无论好坏。那里我们将会展示组件如何可以被测试。

这个蛋糕模式例子实质上很简单。在真是的应用中，我们可能需要一些组件依赖于其他组件，而那些组件有拥有各自的依赖。在这些情况下，事情会变得很复杂。我们将会本书的后续部分更好更详细的展示这一点。

# 本章总结

本章我们探讨了 Scala 中的 AOP，现在我们知道如何将那些本不需要出现在模块中的代码进行拆分。这将有效减少代码重复并使我们的程序拥有不同的专用模块。

我们同样展示了如何使用本书前面部分讨论的技术来创建可复用的组件。组件提供接口并拥有指定的需求，这可以很方便的使用 Scala 的丰富特性。这与设计模式很相关，应为它们拥有相同的目标——使代码更好，避免重复，易于测试。

本书的后续章节我们将会讨论一些具体的设计模式，及其有用特性和用法。我们将会以创建型模式开始，它们由四人帮(Gof)创建，当然，这里会以 Scala 的视角。