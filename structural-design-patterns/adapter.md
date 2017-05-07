---
typora-copy-images-to: ../assets
---

# 适配器模式

在很多情况下，我们需要通过将不同的组件组合在一起来让应用跑起来。然而，通常会出现不同组件的接口互相不兼容的问题。类似于使用一些公共库或任何库，我们自己并不能对其进行修改——因为别人的意见跟我们目前的配置完全一样是不太常见的。这也就是适配器的用途所在。它的目的在于在不修改源代码的情况下帮助兼容各个接口以能够在一起工作。

我们将会在下面的小节中通过类图和实例来展示适配器是如何工作的。

## 类图

对于适配器的类图来说，让我们假如我们需要在应用中转换为使用一个新的日志库。我们尝试使用的新的库拥有一个`log`方法，它接收日志的消息和严重程度。然而，贯穿我们的整个应用，我们期望拥有`info/debug/warning/error`方法并且只接收日志消息，它能够自动的设置严重程度。当然，我们不能编辑原始库的代码，因此需要使用适配器模式。下面的图示展示了适配器模式的类图：

![970FE9DA-DDF7-4E80-A847-B58353E71914](/assets/adapter.png)

在上面的类图中，可以发现我们的适配器 AppLogger 扩展了 Logger，并使用其实例作为一个字段。在实现这些方法的时候，我们就可以简单的调用其`log`方法并传入不同的参数。这也就是通用的适配器实现方式，后面的小节我们会看到具体的代码实例。可能有些情况下不能直接扩展(比如原始类为 final)，我们将展示 Scala 如何来处理这种问题。同时，我们也会展示一些推荐的用法来使用语言特性实现适配器模式。

## 实例

首先，首先看一下我们不能进行修改的`Logger`类的代码：

```scala
class Logger {
  def log(message:String, sevrvity:String):Unit = {
    System.out.println(s"${severity.toUpperCase}: $message")
  }
}
```

我们尽量保持简单以避免读者从本书的主旨上分心。下一步，我们可以要么仅仅编写一个类来扩展`Logger`，或者要么提供一个接口用于抽象。让我们使用第二种方式：

```scala
trait Log {
  def into(message:String):Unit
  def debug(message:String):Unit
  def warning(message:String):Unit
  def error(message:String):Unit
}
```

最终，我们可以创建`AppLogger`：

```scala
class AppLogger extedns Logger with Log{
  overrride def log(messgae:String):Unit = log(message, "info")
  override def warning(message:String):Unit = log(message, "warning")
  override def error(message:String):Unit = log(message, "error")
  override def debug(message:String):Unit = log(message, "debug")
}
```

然后在我们的程序中使用：

```scala
object AdapterExample extends App {
  val logger: Logger = new AppLogger
  logger.info("This is an info message.") 
  logger.debug("Debug something here.") 
  logger.error("Show an error message.") 
  logger.warning("About to finish.") 
  logger.info("Bye!")
}
```

> 你会发现我们并没有完全按照类图的结构实现。我们并不需要  Logger 类的实例作为我们类的一个字段，因为我们的类已经是 Logger 的一个实例了(继承自 Logger)，因此可以直接方法它的 log 方法。

这就是实现和使用基本的适配器模式的方法。然而，有些情况下我们想要适配的类被声明为 final，因此不能再对其进行扩展。下面的小节将展示如何处理这种情况。

### final 类的适配器模式

如果我们声明原始日志类为 final，会发现之前实现的代码将编译错误。这种场景下有另一种适配器方式：

```scala
class FinalAppLogger extends Log {
  private val logger = new FinalLogger
  
  override def info(message: String): Unit = logger.log(message, "info")
  override def warning(message: String): Unit = logger.log(message, "warning")
  override def error(message: String): Unit = logger.log(message, "error")
  override def debug(message: String): Unit = logger.log(message, "debug")
}
```

这里，我们简单的将最终日志类的实例包装在一个类中，并调用它的方法来传入不同的参数。这种用法跟前面的实现完全一样。或者有一种变种是将最终日志类的实例作为构造器参数传入适配日志类。这在原始日志类的创建过程需要一些额外的参数时比较有用。

### Scala 方式的适配器模式

之前我们已经提到过很多次，Scala 是一个特性丰富的语言。基于这样的事实，我们可以使用隐式类来实现适配器的功能。这里我们会使用前面例子中的定义的 FinalLogger 来演示。

隐式类会在可能的地方提供隐式转换。为了能使隐式转换生效，我们需要导入这些隐式声明，这也就是为何它们通常被声明在对象(object)或包对象(package-object)中。对于这个例子，我们将使用包对象：

```scala
package object adapter {
  implicit class FinalLoggerImplicit(logger:FinalLogger) extends Log{
    override def info(message: String): Unit = logger.log(message, "info")
    override def warning(message: String): Unit = logger.log(message, "warning")
    override def error(message: String): Unit = logger.log(message, "error")
    override def debug(message: String): Unit = logger.log(message, "debug")
  }
}
```

这是一个我们日志实例定义位置的包对象。它户自动将`FinalLogger`实例转换为我们的隐式类。下面的代码片段展示了如何使用它：

```scala
object AdapterImplicitExample extends App{
  val logger = new FinalLogger
  logger.info("This is an info message.")
  logger.debug("Debug something here.")
  logger.error("Show an error message.")
  logger.warning("About to finish.")
  logger.info("Bye!")
}
```

## 优点

适配器模式可以用于代码已经被设计并编写。它可以是接口协同工作或进行兼容。其实现和用法也很直接。

## 缺点

上面例子中的最后一种实现存在一些问题。就是每当我们需要使用这个日志器的时候总是需要导入包或对象。同时，隐式类或转换可能会让代码难以阅读和理解。隐式类也存在一些限制，描述在这：http://docs.scala-lang.org/overviews/core/implicit-classes.html。

像我们已经提到过的，当我们有一些无法改变的代码时该模式是有帮助的。如果我们能够修改原始代码，这将是最好的选择，因为在贯穿整个应用中使用适配器模式将会导致程序难以维护和理解。

