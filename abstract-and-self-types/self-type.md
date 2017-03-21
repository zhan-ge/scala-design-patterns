# 自类型

好代码的其中一个特性是关注点分离。开发者需要致力于使类与其方法的职责仅有一个。这有助于测试和维护，而且更有助于简化代码的理解。记得，*简单的总是最好的*。

然而，在编写实际软件时总是无法避免，为了实现某些功能我们需要在一个类的实例中持有别的类的实例。换句话说，一旦我们的对构件块进行了清晰的分离，为了实现功能它们之间则会存在依赖。我们这里所讨论的总结起来实际上就是依赖注入。自类型提供了一种方法来以更加优雅的方式来处理这些依赖。本节中，我们将讨论它们的用法及优点。

## 使用自类型

自类型支持我们在应用中更简便的拆分代码，然后再在其他地方指定那些需要的部分。例子会让一切变得清晰，因此让我们看一个实例。假如我们想要往数据库持久化信息：

```scala
trait Persister[T] {
  def persist(data:T)
}
```

`persist`方法会对数据做一些转换并存储到我们的数据库。当然，我们的代码写的很好，因此数据库实现是互相分离的。我们拥有以下几种数据库：

```scala
trait Database[T] {
  def save(data: T)
}

trait MemoryDatabase[T] extends Database[T] {
  val db:mutable.MutableList[T] = mutable.MutableList.empty
  override def save(data:T):Unit = {
    println("Saving to in memory database.")
    db.+=:(data)
  }
}

trait FileDatabase[T] extends Database[T] = {
  override def save(data:T):Unit = {
    println("Saving to file.")
  }
}
```

我们拥有一个特质及一些具体的数据库实现。那么如何把数据库传递给`Persister`呢？它应该能够调用数据库中的`save`方法。可能会有以下几种方式：

- 在`Persister`中扩展`Database`。这样虽然可行，但是会让`Persister`也变成了`Database`的实例，我们并不希望这样。后面会解释原因。
- 在`Persister`中拥有一个`Database`变量，然后使用它。
- 使用自类型。

为了能够观察自类型是如何工作的，因此使用自类型的方式。我们的`Persister`接口将会变成下面这样：

```scala
trait Persister[T] { this: Database[T] =>
  def persist(data:T):Unit = {
    println("Calling persist.") 
    save(data)
  }
}
```

现在我们访问了数据库中的方法并在`Persister`之内调用了`save`方法。

> **为自类型命名**
>
> 在上面的代码中，我们使用`this: Database[T] =>`语句将自类型包括进来。这样支持我们像使用自身的方法一样直接使用被引入类型的方法。另一种替代的方式是使用`self: Database[T] =>`。有很多例子中使用了后面的方式，这样可以避免当我们需要在嵌套的特质或类定义中使用`this`而引起的混乱。然而这种方式需要在调用被注入的方法时使用`self`来引用。

自类型会要求任何混入`Persister`类同时混入`Database`，否则编译将会失败。让我们创建一些持久化到内存和数据库的类：

```scala
class FilePersister[T] extends Persister[T] with FileDatabase[T]
class MemoryPersister[T] extends Persister[T] with MemoryDatabase[T]
```

最终，我们可以在应用中使用它们：

```scala
object PersisterExample extends App {
  val fileStringPersister = new FilePersister[String]
  val memoryIntPersister = new MemoryPersister[Int]
  
  fileStringPersister.persist("something")
  fileStringPersister.persist("something else")
  
  memoryIntPersister.persist(100)
  memoryIntPersister.persist(123)
}
```

自类型与继承所做的事是不同的，它需要一些代码的存在，因此也能支持我们对功能进行很好的拆分。这可以很大的改变的对程序的维护、重构和理解。

### 使用多个组件

在真实的应用中，可能需要使用自类型来对多个组件做出要求。让我们在例子中展示一个`Histoty`特质，它能够追踪改变并回滚到某个点。不过这里仅做一些打印：

```scala
trait History {
  def add():Unit = {
    println("Action added to history.")
  }
}
```

我们需要在`Persister`中使用它，看起来像是这样：

```scala
trait Persister[T] { this: Database[T] with History =>
  def persist(data:T):Unit = {
    println("Calling persist.")
    save(data)
    add()
  }
}
```

我们可以通过`with`关键字同时添加多个需求。然而，如果我们仅让代码做出这些改变，它并不会编译成功。原因是现在我们必须同时混入`History`到`Persister`中：

```scala
class FilePersister[T] extends Persister[T] with FileDatabase[T] with History
class MemoryPersister[T] extends Persister[T] with MemoryDatabase[T] with History
```

然后再次运行代码，将会得到如下输出：

```
Calling persist.
Saving to file.
Action added to history. Calling persist.
Saving to file.
Action added to history. Calling persist.
Saving to in memory database. Action added to history. Calling persist.
Saving to in memory database. Action added to history.
```

### 组件冲突

在上面的例子中，我们拥有一个对`History`特质的需要，它拥有一个`add`方法。如果不同组件中的方法拥有相同的签名会怎样呢？让我们试一下：

```scala
trait Mystery {
  def add(): Unit = {
    println("Mystery added!")
  }
}
```

然后使用到`Persister`中：

```scala
trait Persister[T] { this:Database[T] with History with Mystery =>
  def persist(data:T):Unit = {
    println("Calling persist.")
    save(data)
    add()
  }
}

class FilePersister[T] extends Persister[T] with FileDatabase[T] with History with Mystery 
class MemoryPersister[T] extends Persister[T] with MemoryDatabase[T] with History with Mystery
```

如果我们允许这个应用，将会得到如下错误信息：

```
Error:(47, 7) class FilePersister inherits conflicting members:
	method add in trait History of type ()Unit and
	method add in trait Mystery of type ()Unit
(Note: this can be resolved by declaring an override in class FilePersister.)
class FilePersister[T] extends Persister[T] with FileDatabase[T] with History with Mystery
^
Error:(48, 7) class MemoryPersister inherits conflicting members:
	method add in trait History of type ()Unit and
	method add in trait Mystery of type ()Unit
(Note: this can be resolved by declaring an override in class MemoryPersister.)class MemoryPersister[T] extends Persister[T] with MemoryDatabase[T] with History with Mystery
							   ^
```

幸运的是这个错误消息已经包含了一些如何修复问题的信息。这跟我们一开始使用特质时遇到的完全是相同的问题，我们可以使用如下的方式修复：

```scala
class FilePersister[T] extends Persister[T] with FileDatabase[T] with History with Mystery{
  override def add():Unit = super[History].add()
}
class MemoryPersister[T] extends Persister[T] with MemoryDatabase[T] with History with Mystery {
  override def add(): Unit = super[Mystery].add()
}
```

然后再次运行例子，将会得到预期的输出：

```
Calling persist.
Saving to file.
Action added to history. Calling persist.
Saving to file.
Action added to history. Calling persist.
Saving to in memory database. Mystery added!
Calling persist.
Saving to in memory database. Mystery added!
```

### 自类型与蛋糕模式

上面我们看到的例子都是单纯的依赖注入的例子。我们通过自类型要要求一个组件必须引入一些指定的组件。

> 自类型常用于依赖注入。他们是*蛋糕设计模式*的主要部分，本书后续部分我们将详细介绍。

蛋糕模式的实现完全依赖于自类型。它鼓励工程师编写小而简单的组件，然后声明并使用它们的依赖。当应用中所有的组件都编写完成后，可以在一个通用的组件中实例化它们以应用于实际的应用。蛋糕模式的一个好的优势是实际上会在编译时来检查所有的依赖是否都满足了。

在本书的后续部分，我们将使用一整个小节来讨论蛋糕模式，那里我们将讨论更多关于该模式是如何被连接在一起的细节，它的优势及缺陷，等等。

## 自类型与继承对比

在上一节中，我们讲到不希望使用继承的方式来访问`Database`的方法。这又是为何呢？如果我们让`Persister`扩展`Database`，这意味着`Persister`本省也变成了一个`Database`(is-a 关系)。然而这是不正确的。这里它只是*使用*一个数据库来实现其功能，而不能称为一个数据库。

继承将子类暴露给父级的实现细节。然而并非总是期望得到这样的结果。根据*Design Patterns: Elements of Reusable Object-Oriented Software*一书的作者所提倡的，开发者总是应该优先使用组合，而不是继承。

### 继承泄露了功能

如果我们使用了继承，同样会将我们不希望的功能泄露给子类。让我们看一下下面的代码：

```scala
trait DB {
  def connect():Unit = println("Connected.")
  def dropDatabase():Unit = println("Dropping!")
  def close():Unit = println("Closed.")
}
trait UserDB extends DB{
  def createUser(username:String):Unit = {
    connect()
    try {
      println(s"Creating a user: $username")
    } finally{
      close()
    }
  }
  
  def getUser(username:String):Unit = {
    connect()
    try{
      println(s"Getting a user: $username")
    } finally {
      close()
    }
  }
}

trait UserService extends UserDB{
  def bad():Unit = dropDatabase()
}
```

这会是一个真实的情况。因为这就是继承的工作方式，我们可以在`UserService`中访问`dropDatabase`。这是一些我们不希望发生的事情，而且可以通过自类型来修复。特质`DB`不需要变动，仅需要修改以下内容：

```scala
trair UserDB{ this:DB =>
  def createUser(username:String):Unit = {
    connect()
    try{
      println(s"Creating a user: $username")
    } finally close()
  }
  def getUser(username:String):Unit = {
    connect()
    try{
      println(s"Getting a user: $username")
    } finally close()
  }
}

trait UserService{ this: UserDB =>
  //...
}
```

这样，在`UserService`中就无法再访问到`dropDatabase`了。我们只能调用我们需要的方法，这也就是我们要真正实现的。