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

TODO