# 特质

大家或许对 Scala 中的特质持有不同的看法。他们不仅可以被视作类似其他语言中的接口，同时可以被当做一个拥有无参构造器的类。

> **特质参数**
>
> Scala 编程语言充满活力，其发展很快。其中一个将会在 2.12 版本中探讨的趋势是特质参数。更多信息请查看 [http://www.scala-lang.org/news/roadmap-next/](http://www.scala-lang.org/news/roadmap-next/)。

在下面的几个小节中，我们将会以不同的视角来了解特质并为你提供一些如何使用它们的想法。

## 特质作为接口

特质可以看做是其他语言中的接口，比如 Java。但是支持开发者实现其中的一部分或全部方法。每当特质中拥有一些代码，该特质则被称为 **mixin(混入、混合)**。让我们看一下下面这个例子：

```scala
trait Alarm {
  def trigger(): String
}
```

这里`Alarm`是一个接口。仅拥有一个`trigger`方法，不拥有任何实现，如果它被混入一个非抽象的类则必须提供一个该方法的实现。

让我们看一下另一个特质的例子：

```scala
trait Notifier {
  val notificationMessage: String
  
  def printNotification(): Unit = {
    println(notificationMessage)
  }
  
  def clear(): Unit
}
```

这个`Notifier`特质拥有一个已实现的方法，而`notificationMessage`的值和`clear`则需要由混入它的类来提供实现。此外，特质可以要求一个类必须拥有一个指定的变量，这类似于其他语言中的抽象类。

### 混入带有变量的特质

像上面提到的，特质可以要求类拥有一个指定的变量。一个有趣的用法是我们可以传递一个变量到类的构造器中。这样则可以满足该特质的需求：

```scala
class NotifierImpl(val notificationMessage:String) extends Notifier {
  override def clear(): Unit = println(""cleared)
}
```

这里唯一的必要是在类的定义中该变量必须使用相同的名字并且前面要使用`val`关键字来修饰。如果上面的代码中我们不适用`val`关键字作为前缀，编译器则仍然会要求我们实现该特质。这种情况下，我们不得不为类的参数使用一个不同的名字，并在类体中拥有一个`override val notificationMessage`赋值。

这种行为的原因很简单：如果我们显式的使用`val`(或 var，根据特质要求)，编译器会创建一个字段，并拥有与参数相同作用域的 getter。如果我们仅拥有参数，仅当该参数被用作构造器的作用域之外时(比如用在该类的方法中)才会创建一个字段和内部的 getter。更完备的是，样例类(case class)会自动在参数之前追加`val`关键字。因此，如果我们使用这个`val`关键字，我们将会自动获得一个与该参数同名的字段、相同的作用域，同时将会自动覆写特质中对我们的要求。

## 特质作为类

特质同样可以以类的视角来看待。这种情况下，他们需要实现其所有的方法，同时仅拥有一个不接收任何参数的构造器。考虑下面的代码：

```scala
trait Beeper {
  def beep(times: Int): Unit = {
    assert(times >= 0)
    1 to times foreach(i => println(s"Beep number: $i"))
  }
}
```

现在我们实际上可以直接实例化`Beeper`并调用其方法。比如下面的例子：

```scala
object BeeperRunner extends App{
  val TIMES = 10
  
  val beeper = new Beeper{}		// <= 实例化一个特质
  beeper.beep(TIMES)
}
```

像预期一样，运行代码后我们将看到如下输出：

```bash
Beep number: 1 
Beep number: 2 
Beep number: 3 
....
```

## 扩展类

特质同样可以用来扩展类。让我们看一下下面的例子：

```scala
abstract class Connector {
  def connect()
  def close()
}

trait ConnectorWithHelper extends Connector {
  def findDriver(): Unit = {
    println("Find driver called.")
  }
}

class PgSqlConnector extends ConnectorWithHelper {
  override def connect(): Unit = {
    println("Connected...")
  }
  
  override def close(): Unit = {
    println("Closed...")
  }
}
```

和预期的一样，`PgSqlConnector`会被抽象类约束来实现其抽象方法。你可能会推测，我们可以用一些别的特质来扩展这些类然后再将他们(同时)进行混入。在 Scala 中，我们会对一些情况稍加限制，在后续章节中我们研究组合的时候会来看一下这么做会对我们产生哪些影响。

## 扩展特质

特质可以被互相扩展。看一下如下例子：

```scala
trait Ping {
  def ping():Unit = {
    println("ping")
  }
}

tring Pong {
  def pong():Unit = {
    println("pong")
  }
}

trait PingPong extends Ping with Pong {
  def pingPong():Unit = {
    ping()
    pong()
  }
}

object Runner extends App with PingPong {
  pingPong()
}
```

> 上面这个例子是比较简单的，实际上也可以让`Runner`分别来扩展两个特质。扩展特质可以很好的用来实现**特质堆叠**这种设计模式，本书的后续部分中我们将会探讨这个话题。

