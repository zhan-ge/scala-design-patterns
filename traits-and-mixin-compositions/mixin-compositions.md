# 混入组合

Scala 支持开发者在一个类中扩展多个特质。这相对于那些不允许同时扩展多个类的语言增加了多重继承的可能性并且能剩下很多编写代码的精力。在这个子主题中，我们将会展示如果将多个特质混入到一个指定的类，或者在编写代码的时候如何创建一个带有某些指定功能的匿名类。

## 混入特质

首先，我们先修改一下上个例子的代码。这个改动很小，它将会展示特质是如何被混入的：

```scala
object MixinRunner extends App with Ping with Pong {
  ping()
  pong()
}
```

从上面的代码中可以发现，我们可以将多个特质添加到同一个类中。上面代码中扩展的`App`特质主要是为了使用其自带的`main`方法。这会和创建无参类有点类似。

> **如何混入特质？**
>
> 将特质混入到类可以使用如下语法：`extends with T1 with T2 with T3 with ... Tn`。
>
> 如果一个类已经扩展了别的类，我们仅需要通过`with`关键字来添加特质。
>
> 如果特质方法在特质体内并没有被实现，同时我们混入该特质的类也没有声明为抽象类，则该类必须实现特质中的方法。否则，将会出现编译错误。

## 组合

创建时的组合给我们提供了一个无需显式定义就能创建匿名类的机会。同样，如果我们想要组合多个不同的特质，创建所有的可能则要花费很多的工作。

### 组合简单特质

让我们看一个组合简单特质的例子，这里不会扩展其他类或特质：

```scala
class watch(brand:String, initialTime: Long){
  def getTime(): Long = System.currentTimeMillis() - initialTime
}

object WatchUser extends App{
  val expensiveWatch = new Watch("expensive brand", 1000L) with Alarm with Notifier {
   override def trigger(): String = "The alarm was triggered."
   override def clear(): Unit = println("Alarm cleared.")
   override val notificationMessage:String = "Alarm is running!"
  }
  val cheapWatch = new Watch("cheap brand", 1000L) with Alarm {
    override def trigger(): String = "The alarm was triggered."
  }
  // show some watch usage.
  println(expensiveWatch.trigger())
  expensiveWatch.printNotification()
  println(s"The time is ${expensiveWatch.getTime()}.")
  expensiveWatch.clear()
  
  println(cheapWatch.trigger())
  println("Cheap watches cannot manually stop the alarm...")
}
```

在这个例子中我们使用了之前定义的`Alarm`和`Notifier`特质。我们创建了两个手表实例——一个是贵的，它拥有更多有用的功能；另一个是便宜的，它的功能则会少很多。本质上，他们都是匿名类，在初始化的时候被定义。另外要注意的是，和预期一样，我们需要实现那些我们扩展的特质中的抽象方法。希望这个例子能为你在拥有很多特质及多种可能的组合时带来一些想法。

只是为了完整，下面是上个程序的输出：

```bash
The alarm was triggered.
Alarm is running!
The time is 1234567890562.
Alarm cleared.
The alarm was triggered.
Cheap watches cannot manually stop the alarm...
```

和预期一样，第三行的时间会在每次运行的时候都有所不同。

### 组合复杂特质

 在有些可能的情况下，我们需要组合更加复杂的特质，这些特质可能扩展了一些别的类或特质。如果在继承链的上层没有一个特质或别的特质已经显式扩展了一个指定的类，事情则会变得很简单，他们也不会改变太多。这种情况下，我们可以简单的访问超特质的方法。然而，让我们看一下如果继承链中的特质已经扩展了一个指定的类会发生什么。在这个例子中，我们将会使用之前定义的`ConnectorWithHelper`特质。该特质扩展了名为`Connector`的抽象类。加入我们想拥有另一个非常昂贵的手表，比如它可以连接到数据库：

```scala
object ReallyExpensiveWatchUser extends App{
  val reallyExpensiveWatch = new Watch("really expensive brand", 1000L) with ConnectorWithHelper {
    override def connect(): Unit = println("Connected with another connector.")
    override def close(): Unit = println("Closed with another connector.")
  }
  
  println("Using the really expensive watch.")
  reallyExpensiveWatch.findDriver()
  reallyExpensiveWatch.connect()
  reallyExpensiveWatch.close()
}
```

看起来都很好，但是当我们编译的时候，会得到如下错误信息：

```bash
Error:(36, 80) illegal inheritance; superclass Watch
	is not a subclass of the superclass Connector of the mixin trait ConnectorWithHelper
	val reallyExpensiveWatch = new Watch("really expensive brand",
	1000L) with ConnectorWithHelper {
^
```

该错误消息告诉我们，由于`ConnectorWithHelper`特质已经扩展了`Connector`类，所有使用该特质进行组合的类必须是`Connector`的子类。现在让我们假如需要混入另一个同样已经扩展了一个类的特质，但被扩展的这个类与之前不同。根据之前的逻辑，会需要`Watch`同样需要是该类的子类。但这是不可能出现的，因为我们同时只能扩展一个类，这也就是 Scala 如何来限制多重继承以避免危险错误的发生。

如果我们想要修复这个例子的编译错误，我们不得不去修改原有的`Watch`类的实现以确保其是`Connector`的子类。然而这可能并非我们所原本期望的，或许这种情况下需要一些重构。

### 组合自类型(self-type)

在上一节中，我们看到了如何在`Watch`类中扩展`Connector`以便能够编译我们的代码。有些场景中我们或许真的需要强制一个类来混入一个特质，或者同时有其他的特质或多个特质需要混入。让我们加入需要一个闹钟同时带有提醒功能：

```scala
trait AlarmNotifier {
  this: Notifier =>
  
  def trigger(): String
}
```

这里我们展示了什么是自类型。第二行代码将`Notifier`的所有方法引入到了新特质的当前作用域，它同时要求所有混入了该特质的类必须同时混入`Notifier`。否则将会出现编译错误。如果使用`self`来代替`this`，我们则可以使用手动的方式来在`AlarmNotifier`中引用`Notifier`的方法，比如`self.printNotification()`。

下面的代码展示了如何来使用这个新的特质：

```scala
object SelfTypeWatchUser extends App {
  AlarmNotifier {
    val watch = new Watch("alarm with notification", 1000l) with AlarmNotifier with Notifier {
      override def trigger():String = "Alarm triggered."
      override def clear(): Unit = println("Alarm cleared.")
      override val notificationMessage:String = "The notification."
    }
  }
  println(watch.trigger())
  watch.printNotification()
  println(s"The time is ${watch.getTime()}.")
  watch.clear()
}
```

如果在上面的代码中去掉同时扩展`Notifier`的部分则会出现编译错误。

在这个小节中，我们展示了子类型的简单用法。一个特质可以要求在被混入的同时混入其他一个或多个特质，多个的时候仅需要使用`with`关键字分割即可。子类型是实现“蛋糕模式”的关键，该模式用于依赖注入。本书后续部分我们会看到更多有趣的用法。

## 特质冲突

你的脑海中可能已经出现了一个问题——如果我们混入的特质中拥有相同签名的方法会怎样？下面的几个小节我们将会探讨该问题。

### 相同签名和返回类型

考虑一个例子，我们想要混入的两个特质拥有相同的方法：

```scala
trait FormalGreeting {
  def hello():String
}

trait InformalGreeting {
  def hello():String
}

class Greeter extends FormalGreeting with InformalGreeting {
  override def hello():String = "Good morning, ser/madam!"
}

object GreeterUser extends App {
  val greeter = new Greeter()
  println(greetrt.hello())
}
```

在这个例子中，接待员总是会很有礼貌并且同时混入正式的和非正式的问候。在实现时仅需要实现该方法一次。

### 相同签名和不同返回类型

如果我们的问候特质拥有更多方法，签名相同但返回类型不同呢？我们将下面的声明添加到`FormalGreeting`中：

```scala
def getTime():String
```

同时向`InformalGreeting`中添加：

```scala
def getTime():Int
```

这种情况下我们需要在`Greeter`中实现同时实现这两个方法。然而，编译器不允许我们定义`getTime`两次，这表示 Scala 中会避免发生这样的事。

### 相同签名和返回类型的混入

在继续之前，快速回忆一下混入只是一个带有一些代码实现的特质。这意味着在下面的例子中，我们不需要在使用它们的类中实现这些方法：

```scala
trait A {
  def hello(): String = "Hello, I am trait A!"
}

trait B {
  def hello(): String = "Hello, I am trait B!"
}

object Clashing extends App with A with B {
  println(hello())
}
```

可能和预期一样，我们会得到一个编译错误信息：

```bash
Error:(11, 8) object Clashing inherits conflicting members:
	method hello in trait A of type ()String and
	method hello in trait B of type ()String
(Note: this can be resolved by declaring an override in object Clashing.) 
object Clashing extends A with B { 
       ^
```

该信息很有用，它甚至为我们提供了一个如何修复问题的提示。方法冲突在多重继承中是一个问题，但是和你看到的一样，我们致力于选择一个可用的方法。在`Clashing`对象中我们或许可以这样修改：

```scala
override def hello():String = super[A].hello()
```

然而，如果处于某些原因我们相同时使用两个方法呢？这种情况下，我们可以创建另外一个名字不同的方法来调用另一个指定特质中的方法。我们同样可以直接通过`super`符号直接引用这些方法而不是将他们包装在另一个方法中。然而我个人更倾向于包装在另一个方法内，否则代码将会变得很乱。

> **super 符号**
>
> 如果在上面的例子中，我们直接使用`override def hello(): String = super.hello()`而不是`super[A]. hello()`，真正被选择的又是那个特质中的方法呢？这种情况下将会选择 B 中的方法。这取决于 Scala 中的线性化特性，我们将在后面的章节中详细讨论。

### 相同签名和不同返回类型的混入

和预期一样，如果方法的传入参数在类型或数量上有所不同则上面的问题就不再存在，因为这成了一个新的签名。但如果特质中有如下两个方法问题则仍然存在：

```scala
def value(a: Int): Int = a // in trait A 
def value(a: Int): String = a.toString // in trait B
```

我用用过的方式在这里不再有效，你可能会对此感到吃惊。如果我们决定仅覆写特质 A 中的方法，将会得到如下编译错误：

```scala
Error:(19, 16) overriding method value in trait B of type (a: Int): String; 
  method value has incompatible type 
  override def value(a: Int): Int = super[A].value(a)
  			   ^
```

如果重写 B 中的方法，错误也会随之改变。而如果两个都覆写，则会得到如下错误：

```scala
Error:(20, 16) method value is defined twice 
  conflicting symbols both originated in file '/path/to/traits/src/main/ scala/com/ivan/nikolov/composition/Clashing.scala'
  override def value(a: Int): String = super[B].value(a)
```

这展示了 Scala 会避免我们在多重继承中进行这样危险的操作。为了完整，如果你遇到类似的问题，仍然存在变通的方式，比如像下面的例子一样，牺牲掉混入的功能：

```scala
trait D {
  def value(a:Int):String = a.toString
}

object Example extends App{
  val c = new C{}
  val d = new D{}
  
  println(s"c.value: ${c.value(10)}")
  println(s"d.value: ${d.value(10)}")
}
```

这段代码中把特质当做合作者使用，但这也丢掉了混入这些特质的类的实例同样也拥有这些特质的类型这一事实(即扩展了特质的类，其实例同时拥有特质的类型)，这一性质在某些操作中会很有用。

