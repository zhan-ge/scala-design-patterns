---
typora-copy-images-to: ../assets
---

# 工厂方法设计模式

工厂方法模式用于封装实际类的初始化。它简单的提供一个借口来创建对象，然后工厂的子类来决定创建哪个具体的类。当我们需要在应用的运行时创建不同类的实例，这种模式将会很有用。这种模式同样可以有效的用于当需要开发者传入额外的参数时。

例子可以让一切变得简单，后面的小节我们将会提供一个实例。

## 类图

对于工厂方法，我们将会展示一个数据库相关的例子。为了使事情变得简单(因为实际的`java.sql.Connection`拥有很多方法)，我们将会定义自己的`SimpleConnection`，它会拥有 MySQL 和 PostgreSQL 的具体实现。

这个连接类的类图看起来会像下面这样：

![SimpleConnection-diagram](/assets/SimpleConnection-diagram.png)

现在，连接的创建将会基于我们选择使用的数据库。然而，因为它们提供的接口，用法将会完全一样。实际的创建过程可能需要执行一些额外的、我们想要对用户隐藏的参数计算，而如果我们对每个数据库使用不同的常数，这些计算也会是相关的。这也就是为什么我们要使用工厂方法模式。下面的图示展示了剩余部分的代码将会如何被组织：

![database-client-diagram](/assets/database-client-diagram.png)

上面的图示中，`MysqlClient`和`PgsqlClient`都是`DatabaseClient`的具体实现。工厂方法`connect`将会在不同的客户端中返回不同的实际连接。即便我们进行了覆写，代码中的签名仍然会显示返回一个`SimpleConnection`，但实际上会是具体类型。在这个类图中，为了清晰我们选择展示实际的返回类型。

## 代码实例

基于上面类图中清晰的展示，基于我们要使用的数据库客户端，一个不同的连接将会被创建和使用。让我们看一下这个类图的代码表示。首先是`SimpleConnection`以及其具体实现：

```scala
trait SimpleConnection{
  def getName():String
  def executeQuery(query:String):Unit
}

class SimpleMysqlConnection extends SimpleConnection{
  override def getName():String = "SimpleMySQLConnection"
  override def executeQuery(query:String):Unit = {
    System.out.println(s"Executing the query '$query' the MySQL way.")
  }
}

class SimplePgSqlConnection extends SimpleConnection {
  override def getName():String = "SimpePgSqlConnection"
  override def executeQuery(query:String):Unit = {
    System.out.println(s"Executing the query '$query' the PgSQL way.")
  }
}
```

对于这些实现的使用将会发生在工厂方法中，名为`connect`。下面的代码片段展示了我们可以如何利用连接，以及如何在特定的数据库客户端中实现。

```scala
// the factory
abstract class DatabaseClient {
  def executeQuery(query:String):Unit = {
    val connection = connect()
    connection.executeQuery(query)
  }
  
  // the factory method
  protected def connect():SimpleConnection
}

class MysqlClient extends DatabaseClient {
  override protected def connect():SimpleConnection = new SimpleMysqlConnection
}

class PgSqlClient extends DatabaseClient {
  override protected def connect():SimpleConnection = new SimplePgSqlConnection
}
```

然后可以直接使用这些客户端：

```scala
object Example extends App {
  val clientMysql:DatabaseClient = new MysqlClient
  val clientPgsql:DatabaseClient = new PgSqlClient
  
  clientMySql.executeQuery("SELECT * FROM users")
  clientPgSql.executeQuery("SELECT * FROM employees")
}
```

我们看到了工厂方法模式是如何工作的。如果我们需要添加另一个数据库客户端，则可以扩展`DatabaseClient`，并在实现`connect`方法时返回一个扩展自`SimpleConnection`的类。

> 上面例子中选择使用特质来定义 SimpleConnection，并使用抽象类来实现 DatabaseClient，这只是随机的选择。当然，我们完全可以使用特质来替代抽象类(除非需要参数构造器的时候)。

在有些场景中，由工厂方法创建的对象可能会在构造器中需要一些参数，而这些参数有可能基于持有工厂方法的对象的一些特定状态或功能。这也是该模式的实际闪光点。

## Scala 中的替代选择

与软件工程中的任何事情一样，该模式同样可以由不同的方式来实现。不同的选择则要归结于需求的不同和应用、被创建对象的特定属性。一些可选的替代方法有：

- 通过构造器向该类传入其需要的组件(对象组合)。这样一来可能意味着，每次请求，这些组件可能是特定的实例而非一个新的。
- 传入一个能够创建我们所需要的实例的函数。

使用 Scala 的丰富特性，我们能够避免这种设计模式，我们能够更明智的知道，我们将要使用或暴露的这些对象是如何创建的，而无论工厂方法是如何被创建的。没有绝对对错的方式。然而，应该基于特定的需求来选择一种能够同时满足使用、维护更加简便的方式。

## 优点

和其他工厂一样，对象的创建细节被隐藏了。这意味着，如果我们改变了一个特定对象的创建方式，我们仅需要修改创建它的工厂方法(基于设计，这可能会涉及很多创建器)。工厂方法支持我们使用一个类的抽象版本并把对象的创建指派给子类。

## 缺点

在上面的例子中，如果我们拥有多于一个的工厂方法则很快会遇到问题。这首先需要开发者实现更多的方法，但更重要的是，这将导致返回的对象互不兼容。让我们看一个小例子。首先我们会定义另一个特质`SimpleConnectionPrinter`，拥有一个在被调用时打印一些东西的方法：

```scala
trait SimpleConnectionPrinter {
  def printSimpleConnection(connection:SimpleConnection):Unit
}
```

然后我们想改变我们的`DatabaseClient`并换个不同的名字：

```scala
abstract class BadDatabaseClient {
  def executeQuery(query:String):Unit = {
    val connection = connect()
    val connectionPrinter = getConnectionPrinter()
    connectionPrinter.printSimpleConnection(connection)
    connection.executeQuery(query)
  }
  
  protected def connect(): SimpleConnection
  protected def getConnectionPrinter(): SimpleConnectionPrinter
}
```

与原有例子不同的地方在于我们多了一个工厂方法，并在执行查询的时候同时被调用。类似于`SimpleConnection`的实现，现在让我们为`SimpleConnectionPrinter`创建另外两个具体实现：

```scala
class SimpleMysqlConnectionPrinter extends SimpleConnectionPrinter{
  override def printSimpleConnection(connection:SimpleConnection):Unit = {
    System.out.println(s"I require a MySQL connection. It is: '${connection.getName()}'")
  }
}

class SimplePgSqlConnectionPrinter extends SimpleConnectionPrinter{
  override def printSimpleConnection(connection:SimpleConnection):Unit = {
    SimpleConnection): Unit = { System.out.println(s"I require a PgSQL connection. It is: '${connection.getName()}'")
  }
}
```

现在我们可以应用工厂设计模式来创建 MySQL 和 PostgreSQL 客户端：

```scala
class BadMysqlClient extends BadDatabaseClient {
  override protected def connect():SimpleConnection = new SimpleMysqlConnection
  override protected def getConnectionPrinter(): SimpleConnectionPrinter = 
    new SimpleMySqlConnectionPrinter
}

class BadPgsqlClient extends BadDatabaseClient {
  override protected def connect(): SimpleConnection = new SimplePgSqlConnection
  override protected def getConnectionPrinter(): SimpleConnectionPrinter = 
    new SimpleMySqlConnectionPrinter	// note here
}
```

上面这个实现则有效完成了，现在我们可以使用它：

```
object BadExample extends App{
  val clientMySql: BadDatabaseClient = new BadMySqlClient
  val clientPgSql: BadDatabaseClient = new BadPgSqlClient
  clientMySql.executeQuery("SELECT * FROM users")
  clientPgSql.executeQuery("SELECT * FROM employees")
}
```

但是运行会得到如下结果：

```
I require a MySQL connection. It is 'SimpleMysqlConnection'
Execution the query 'SELECT * FROM users' the MySQL way.
I require a Mysql connection. It is 'SimplePysqlConnection'
Execution the query 'SELECT * FROM employees' the PgSQL way.
```

上面的例子显然发生了一个逻辑错误，同时也没有给我们提供任何提醒(在具体的工厂方法中使用了错误的具体实现)。随着要实现的方法数量的增长，这将会成为一个问题，并且错误会更易于发生。比如，我们的代码没有抛出任何异常，但是这种陷阱会引起运行时错误，这让问题变得难于发现和调试。