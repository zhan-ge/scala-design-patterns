---
typora-copy-images-to: ../assets
---

# 抽象工厂

抽象工厂是工厂模式系列的另一种设计模式。其目的与所有工厂设计模式一样——封装对象的创建逻辑并对用户隐藏。不同在于它的实现方式。

抽象工厂设计模式基于**对象组合**，而非**继承**，像工厂方法模式的实现中则是基于继承。这里我们有一个单独的对象，它提供一个借口来创建我们需要的类的实例。

## 类图

这里我们仍然使用前面的`SimpleConnection`例子。下面的类图展示了抽象工厂是如何被组织的：

![ABB20C6F-A941-43F4-B3FB-0B3829D98511](/assets/abstract-factory-diagram.png)

从上图中我们可以发现，现在我们可以拥有一个带有层级结构的工厂，而非数据库客户端中的一个方法。我们将会在应用中使用`DatabaseConnectorFactory`，同时它也会基于实际的实例类型返回正确的对象。

## 实例

让我们以代码的视角来观察我们的例子。下面的代码清单展示了工厂的层级结构：

```scala
trait DatabaseConnectionFactory{
  def connection():SimpleConnection
}

class MysqlFactory extends DatabaseConnectionFactory{
  override def connect():SimpleConnection = new SimpeMysqlConnection
}

class PgsqlFactory extends DatabaseConnectionFactory{
  override def connect():SimpleConnection = new SimplePgsqlConnection
}
```

然后我们可以将这些工厂传递给类并调用需要的方法来使用它们。这里有一个与之前展示的工厂方法类似的例子：

```scala
class DatabaseClient(connectionFactory:DatabaseConnectionFactory){
  def executeQuery(query:String):Unit = {
    connectionFactory.connect()
    connection.executeQuery(query)
  }
}
```

然后使用这个数据库客户端：

```scala
object Example extends App {
  val clientMysql:DatabaseClient = new DatabaseClient(new MysqlFactory)
  val clientPgsql:DatabaseClient = new DatabaseClient(new PgsqlFactory)
  
  clientMysql.executeQuery("SELECT * FROM users")
  clientPgsql.executeQuery("SELECT * FROM employees")
}
```

这就是抽象工厂的工作方式了。如果我们需要添加数据库客户端，只需要添加一个新的具体工厂来扩展`DatabaseConnectionFactory`。这给重构和扩展带来很大的遍历。

## Scala 中的替代选择

这种设计模式同样可以以不同的方式实现。实际上，我们使用对象组合来将工厂传递给类也暗示着也可以使用别的方式：我们可以简单的传入一个函数，因为在 Scala 中它们也是统一化的一部分，它们会被与对象相同的方式处理。

## 优点

像所有工程模式一样，对象的创建细节被隐藏了。当我们想要暴露一系列对象时(比如数据库连接器)，这种模式尤其有用。这时客户端会与具体类解耦。该模式通常会在一些 UI 工具集中最为示例展示，比如那些基于操作系统的不同而有所不同的元素。它也是测试友好的，因为我们可以给客户端提供一些模拟器而非实际的工厂。

尽管我们前面提过的不兼容问题在这里仍然存在，不过现在会很难遇到。主要是因为这里客户端实际上仅仅会传递一个单独的工厂作为参数。这种情况下我们为用户提供了一个具体的工厂，而编写这些工厂的时候一切都被静心安排过了。

## 缺点

如果我们使用的对象(这里是 SimpleConnection)和方法改变了签名，将会引起问题的出现。有些情况下，该模式可能会为我们的代码带来一些没有必要的复杂性，以及难以阅读和维护。

