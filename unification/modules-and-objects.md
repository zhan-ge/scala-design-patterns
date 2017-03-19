# 模块和对象

模块是组织程序的一种方式。它们是可互换的和可插拔的代码块，有明确定义的接口和隐藏的实现。在 Java 中，模块以包的形式组织。在 Scala 中，模块是对象，就像其他的一切一样。这意味它们可以被参数化、被扩展、像参数一样传递，等等。

Scala 的模块可以提供必要环境以便使用。

## 使用模块

我们已经总结了 Scala 中模块也是对象而且同样被统一化了。这意味着我们可以在我们的应用中传递一个完整的模块。这可以很有用，然而，下面展示一个模块看起来是什么样子：

```scala
trait Tick {
  trait Ticker {
    def count(): Int
    def tick(): Unit
  }
  def ticker:Ticker
}
```

这里，`Tick`仅是我们一个模块的一个接口。下面是它的实现：

```scala
trait TickUser extends Tick {
  class TickUserImpl extends Ticker{
    var curr = 0
    override def cont(): Int = curr
    override def tick():Unit = curr = curr + 1
  }
  object ticker extends TickUserImpl
}
```

这个`TickUser`实际上是一个模块。它实现了`Tick`并在其中隐藏了代码。我们创建了一个单例对象来提供实现。对象的名字和`Tick`中的方法名一样。这样混入该特质的时候则能满足其实现的需求。

类似的，我们可以像下面这样定义一个接口和实现：

```scala
trait Alarm {
  trait Alarmer {
    def trigger(): Unit
  }
  def alarm:Alarmer
}
```

然后是实现：

```scala
trait AlarmUser extends Alarm with Tick{
  class AlarmUserImpl extends Alarmer {
    override def trigger(): Unit = {
      if(trigger.count() % 10 == 0){
        println(s"Alarm triggered at ${ticker.count()}!")
      }
    }
  }
  object alarm extends AlarmUserImpl
}
```

有意思的是我们在`AlarmUser`同时扩展了两个模块。这展示了模块可以怎样互相基于对方。最终，我们可以像下面这样使用这些模块：

```scala
object ModuleDemo extends App with Alarmer with TickUser{
  println("Running the ticker. Should trigger the alarm every 10 times.")
  (1 to 100).foreach {
    case i => 
      ticker.tick()
      alarm.trigger()
  }
}
```

为了让`ModuleDemo`能够使用`AlarmUser`模块，编译器会要求混入`TickUser`或任何混入了`Tick`的任何模块。这提供了一种可能性来插拔一个不同的功能。

下面是程序的输出：

```scala
Running the ticker. Should trigger the alarm every 10 times. Alarm triggered at 10!
Alarm triggered at 20!
Alarm triggered at 30!
Alarm triggered at 40!
Alarm triggered at 50!
Alarm triggered at 60!
Alarm triggered at 70!
Alarm triggered at 80!
Alarm triggered at 90!
Alarm triggered at 100!
```

> Scala 中的模块可以像其他对象一样传递。他们是可扩展的、可互换的，并且实现是被隐藏的。

