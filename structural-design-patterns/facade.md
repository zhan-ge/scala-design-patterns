---
typora-copy-images-to: ../assets
---

# 外观模式

每当我们要构建一些库或大型的系统，总是会需要依赖一些其他的库或功能。方法的实现有时需要同时依赖多个类。这就需要一些必要的知识以了解这些类。无论何时我们为用户构建一些库，我们总是尝试使其对用户来说变得简单以假设他们并不会也并不需要像我们一样拥有广泛的领域知识。另外，开发者需要确保那些组件能够在整个用户的应用中易于使用。这也就是外观模式的用途所在。

> 其目的在于通过一个简单的结构来包装一个复杂的系统，以隐藏使用的复杂性来简化客户端的交互。

我们已经看到过一些基于包装来实现的设计模式。适配器模式通过将一个接口转换为另一个，装饰器用来添加额外的功能，而外观模式则使一切变得更加简单。

## 类图

对于类图，让我们假设一下的设置：我们想要用户能够从服务端下载一些数据并能以对象的方式完成反序列化。服务端以编码的形式返回数据，因此我们首先要对其进行解码，然后解析，最终返回正确的对象。所涉及的这些大量的操作使事情变的复杂。这也就是我们要使用外观模式的原因。

![facade](/assets/facade.png)

当客户端使用上面的应用时，他们仅需要与`DataReader`进行交互。而在内部会处理好对数据的下载、解码及反序列化。

## 实例

上面的类图中展示了`DataDownloader`、`DataDecoder`、`DataDeserializer` 作为对象组合在`DataReader`中使用。这样实现起来直接清晰——它们要么可以通过默认的构造器创建，要么作为参数传入。对于我们例子的代码展示来说，我们选择使用特质来代替类，并将它们混入到`DataReader`类中。

首先让我们看一下这几个特质：

```scala
trait DataDownloader extends LazyLogging {
  def download(url:String):Array[Byte] = {
    logger.info("Downloading from: {}", url)
    Thread.sleep(5000)
    // { 
    // "name": "Ivan", 
    // "age": 26 
    // } 
    // the string below is the Base64 encoded Json above.
    "ew0KICAgICJuYW1lIjogIkl2YW4iLA0KICAgICJhZ2UiOiAyNg0KfQ==".getBytes
  }
}

trait DataDecoder{
  def decode(data: Array[Byte]): String = 
    new String(Base64.getDecoder.decode(data), "UTF-8")
}

trait DataDeserializer{
  implicit val formats = DefaultFormats
  
  def parse[T](data: String)(implicit m: Manifest[T]): T = 
    JsonMethods.parse(StringInput(data)).extract[T]
}
```

上面的这些实现十分简单并且相互分离，因为它们都处理不同的任务。任何人都可以使用它们；然而，这会需要一些必备的知识并使事情变得更加复杂。这也是为什么我们把外观类称为`DataReader`：

```scala
class DataReader extends DataDownloader with DataDecoder with DataDeserializer{
  def readPerson(url:String):Person = {
    val data = download(url)
    val json = decode(data)
    parse[Person](json)
  }
}
```

这个实例清晰的展示了替代使用三个不同的接口，我们可以使用一个简单的方法调用。而所有的复杂性都隐藏在方法的内部。下面的清单展示了这个类的用法：

```scala
object FacadeExample extends App{
  val reader = new DataReader
  System.out.println(s"We just read the following person: ${reader.readPerson("http://www.ivan-nikolov.com/")}")
}
```

当然，在这个例子中，我们同样可以使用类来代替特质混入，这完全基于实现的需要。

## 优点

外观模式可以用于隐藏库的实现细节，使接口更加易于用来交互负载的系统。

## 缺点

人们常犯的一个错误是尝试把一切都放到一个外观中。这种形式是没有任何帮助的，开发者仍然要面对复杂的系统。另外，外观可能会被证明限制了那些拥有足够领域知识的用户对原始的功能的使用。当门面称为与内部系统交互的唯一方式时这种限制就尤为明显。

