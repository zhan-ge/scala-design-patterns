# 单例模式

单例模式用于确保一个类在整个应用中仅有一个实例。它在使用它的应用用引入了一个全局状态。

一个单例对象可以由不同的策略来初始化——懒初始化或热(eager)初始化。这都基于期望的用途，以及对象需要被初始化的时机，等等。

## 类图

单例是另一种由 Scala 语言语法所支持的设计模式。我们通过`object`关键字来实现它，并没有必要提供一个类图，因此我们下一小节直接进入一个实例。

## 实例

这个实例的目的是展示如果在 Scala 中创建一个单例对象，以及理解在 Scala 中对象是何时被创建的。我们将会看到一个名为`StringUtils`的类，提供了一些字符串相关的工具方法：

```scala
object StringUtils{
  def countNumberOfSpecs(text:String):Int = text.split("\\+s").length -1
}
```

这个类的用法也很直接。Scala 会管理对象的创建过程、线程安全，等等：

```scala
object UtilsExample extends App{
  val sentence = "Hello there! I am a utils example."
  println(s"The number of spaces in '$sentence' is:
  ${StringUtils.countNumberOfSpaces(sentence)}")
}
```

尽管`StringUtils`对象是一个单例实例，上面这个例子看起来仍然很清晰，类似于带有静态方法的类。这也就是在 Scala 中定义静态方法的方式。给一个单例类添加状态或许会更有意思。下面的例子展示了这种方式：

```scala
object AppRegistry{
  println("Registry initialization block called.")
  private val users: Map[String, String] = TrieMap.empty
  
  def addUser(id: String, name: String): Unit = { users.put(id, name) }
  def removeUser(id: String): Unit = { users.remove(id) }
  def isUserRegistered(id: String): Boolean = users.contains(id)
  def getAllUserNames(): List[String] = users.map(_._2).toList
}
```

`AppRegistry`包含一个使用应用的当前用户所构成的并发 Map。这是我们的全局状态，同时提供了一些方法来支持用户操作它。同时我们有一个打印语句，它会在这个单例对象被创建时执行。我们可以在下面的应用中使用这个注册表：

```scala
object AppRegistryExample extends App{
  System.out.println("Sleeping for 5 seconds.") 
  Thread.sleep(5000)
  System.out.println("I woke up.")
  AppRegistry.addUser("1", "Ivan")
  AppRegistry.addUser("2", "John") 
  AppRegistry.addUser("3", "Martin") 
  System.out.println(s"Is user with ID=1 registered? ${AppRegistry.isUserRegistered("1")}") 
  System.out.println("Removing ID=2") 
  AppRegistry.removeUser("2") 
  System.out.println(s"Is user with ID=2 registered? ${AppRegistry.isUserRegistered("2")}") 
  System.out.println(s"All users registered are: ${AppRegistry.getAllUserNames().mkString(",")}")
}
```

从运行这段代码得到的输出你会发现，在 “I woke up” 之后会打印单例对象被初始化的信息，因为在 Scala 中，单例会被懒初始化。

## 优点

在 Scala 中，单例模式与静态方法的实现方式一样。这也是单例可以有效的用于创建没有状态的工具类的原因。Scala 中的单例也可以用于创建 ADT。

另一个在 Scala 中严格有效的事情是，单例会被以线程安全的方式创建，因此无需一些额外的操作来确保。

## 缺点

单例模式实际上通常被称为是“anti-pattern”(反面模式、反面教材)。很多人说全局状态不能以单例的方式来实现。也有人说如果你不得不使用单例模式，则你需要尝试重构你的代码。虽然这在有些场景中是对的，但有些场景很适合使用单例模式。通常首要的原则是：如果你能避免，那就避免。

对于 Scala 的单例需要特别指出的一点是它可以真的仅拥有一个实例。虽然这实际上是该模式的定义，在其他语言中，我们可以拥有一个预定义的数量而非仅仅一个单例对象，我们可以使用自定义的逻辑进行控制。

有一点对 Scala 没有影响，不过还是值得一提。当应用中的单例被延迟初始化时，为了能够提供线程安全，需要基于一种加锁机制，比如前面提到过的*double-checked locking*。访问应用中的单例时，无论是在 Scala 中还是别的语言，同样需要以线程安全的方式来完成，或者由单例内部来处理这个问题。

