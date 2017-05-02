# 实例

效率是每个程序中很重要的部分。很多情况下，我们可以对方法计时以查找应用中的瓶颈。让我们看一个示例程序。

我们将会看一下解析。在很多真实的应用中，我们需要以特定的格式读取数据并将其解析为我们代码中的对象。比如，我们拥有一个记录人员的小数据库并以 JSON 格式表示：

```json
[
  {
	"firstName": "Ivan", 
    "lastName": "Nikolov", 
    "age": 26 
  }, 
  { 
    "firstName": "John", 
    "lastName": "Smith", 
    "age": 55 }, 
  { 
    "firstName": "Maria", 
    "lastName": "Cooper", 
    "age": 19 
  }
]
```

为了在 Scala 中表示这段 Json，我们需要定义模型。这会很简单而且只有一个类：Person。下面是代码：

```scala
case class Person(firstName:String, lastName:String, age:Int)
```

由于我们要读取 Json 输入，因此要对其进行解析。有很多解析器，每种或许会有各自不同的特性。在当前这个例子中我们将会使用  Json4s。这需要在`pom.xml`中配置额外的依赖：

```xml
<dependency> 
  <groupId>org.json4s</groupId> 
  <artifactId>json4s-jackson_2.11</artifactId> 
  <version>3.2.11</version> 
</dependency>
```

上面这个依赖可以轻易转换为 SBT，如果读者更愿意使用 SBT 作为构建系统的话。

我们要编写一个拥有两个方法的类，用于解析前面所指定的格式的输入文件并返回一个`Person`对象列表。这两个方法实际上在做同一件事，但是其中一个会比另一个效率更高：

```scala
trait DataReader{
  def readData():List[Person]
  def readDataInefficiently():List[Person]
}

class DataReaderImpl extends DataReader{
  implicit val formats = DefaultFormats
  private def readUnitimed():List[Person] = 
    parse(StreamInput(getClass.getResourceAsStream("/users/json"))).
      extract[List[Person]]

  override def readData():List[Person] = readUntimed()
  
  override def readDataInefficiently():List[Person] = {
    (1 to 10000).foreach{
      case num => readUntimed()
    }
    readUntimed()
  }
}
```

特质`DataReader`扮演一个借口，对实现的使用也十分直接：

```scala
object DataReaderExample extends App{
  val dataReader = new DataReadImpl
  System.out.println(s"I just read the following data efficiently: ${dataReader.readData()}") 
  System.out.println(s"I just read the following data inefficiently: ${dataReader.readDataInefficiently()}")
}
```

运行这段代码将会得到和预期一样的结果。

上面的这个例子是很清晰的。然后，如果你想优化你的代码并查看运行缓慢的原因呢？上面的代码并没有提供这种能力，因此我们要做一些额外的工作来对应用计时并查看它是如何执行的。下一个小节我们将会同时展示不适用 AOP 和使用 AOP 的实现。

## 不使用 AOP

有一种基本的方法来进行计时。我们可以把`println`语句包裹起来，或者让计时称为`DataReaderImpl`类方法的一部分。通常，将计时作为方法的一部分会是一个更好的选择，因为这个方法可能会在不同的地方被调用，同时它们的性能也取决于传入的参数和一些其他因素。基于我们所说，这也就是`DataReaderImpl`类将会如何被重构以支持计时的方式：

```scala
class DataReaderImpl extends DataReader {
  implicit val formats = DefaultFormats
  private def readUnitimed():List[Person] = parse(StreamInput(getClass.getResourceAsStream("users.json"))).extract[List[Person]]
  override def readData(): List[Person] = {
    val startMillis = System.currentTimeMillis()
    val result = readUnitimed()
    val time = System.currentTimeMillis() - startMillis
    System.err.println(s"readData took $time milliseconds")
    result
  }
  
  override def readDataInefficiently():List[Person] = {
    val startMillis = System.currentTimeMillis()
    (1 to 1000) foreach {
      case num => readUntimed()
    }
    val result = readUntimed()
    val time = System.currentTimeMillis() - startMillis
    System.err.println(s"readDataInefficiently took ${time} milliseconds.")
    result
  }
}
```

因此你会发现，代码会变得不可读，计时功能干扰了实际的功能。运行这段代码将会发现其中一个方法花费的更多的时间来执行。

在下节中将会展示如何使用 AOP 来提升我们的代码。

## 使用 AOP

向前面看到的一样，向我们的方法中添加计时代码将会引入重复代码同时也使代码难以追踪，尽管是一个很小的例子。现在，假如我们同样需要打印一些日志或进行一些其他活动。AOP 将会帮助分离这些关注点。

我们可以把`DataReadImpl`重置到一开始的状态，这时它不再打印任何日志。现在创建另一个名为`LoggingDataReader`的特质，它扩展自`DataReader`并拥有以下内容：

```scala
trait LoggingDataReader extends DataReader {
  abstract override def readData(): List[Person] = {
    val startMillis = System.currentTimeMillis()
    val result = super.readData()
    val time = System.currentTimeMillis() - startMillis
    System.err.pringln(s"readData took $time milliseconds.")
    result
  }
  
  abstract override def readDataInefficiently():List[Person] = {
    val startMillis = System.currentTimeMillis()
    val result = super.readDataInefficiently()
    val time = System.currentTimeMillis() - startMillis
    System.err.println(s"readDataIneffieciently took $time milliseconds.")
    result
  }
}
```

这里有趣的地方在于`abstract override`修饰符。它提醒编译器我们会进行**叠加性(stackable)**的修改。如果我们不使用该修饰符，编译将会失败：

```
Error:(9, 24) method readData in trait DataReader is accessed from super. It may not be abstract unless it is overridden by a member declared `abstract' and `override'
	val result = super.readData()
					   ^

Error:(17, 24) method readDataInefficiently in trait DataReader is accessed from super. It may not be abstract unless it is overridden by a member declared `abstract' and `override'
val result = super.readDataInefficiently()
				   ^
```

现在让我们的新特质使用之前提到过的混入组合，在下面的程序中：

```scala
object DataReaderAOPExample extends App{
  val dataReader = new DataReaderImpl with LoggingDataReader
  System.out.println(s"I just read the following data efficiently: ${dataReader.readData()}") 
  System.out.println(s"I just read the following data inefficiently: ${dataReader.readDataInefficiently()}")
}
```

运行这段代码将会得到带有计时信息的输出。

使用 AOP 的优势是很明显的——实现不会被其他代码污染。再者，我们可以以同样的方式添加更多修改——更多的日志、重试逻辑、回滚等等。所有这些都可以通过创建一个新特质并扩展`DataReader`接口，然后在创建具体实现的实例中混入即可。当然，我们可以同时应用多个修改，它们将会按顺序执行，而顺序将会遵循线性化原则。

