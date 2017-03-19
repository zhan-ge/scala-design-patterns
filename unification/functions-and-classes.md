---
typora-copy-images-to: ../assets
---

# 函数与类

在 Scala 中，所有的值都是一个对象。函数作为第一类值，同时作为他们各自的类的对象。下面的图示展示了 Scala 中被统一的类型系统和实现方式。该图来自 [http://www.scala-lang. org/old/sites/default/files/images/classhierarchy.png](http://www.scala-lang. org/old/sites/default/files/images/classhierarchy.png)，它表示了模型的最新视图(有些类比如`ScalaObject`已经被移除)。

又可以发现，Scala 中并没有 Java 中的原始类型概念，所有的类型最终都是`Any`的子类型。

![04666A3B-041D-4DEB-BF44-91A88A496A70](/assets/type-hierarchies.png)

## 函数作为类

函数作为类实际上意味着如果他们仅仅是值的话，则可以被自由的传递给其他方法或类。这提高了 Scala 的表现力也让他相对与其他语言(比如 Java)更易于实现一些事情，比如回调。

### 函数字面值

让我们看一个例子：

```scala
class FunctionLiterals {
  val sum = (a:Int, b:Int) => a + b
}

object FunctionLiterals extends App{
  val obj = new FunctionLiterals
  println(s"3 + 9 = ${obj.sum(3, 9)}")
}
```

这里我们可以看到`FunctionLiterals`类的`sum`字段是如何被赋值为一个函数的。我们可以将任意函数赋值给一个变量，然后把它当做一个函数调用(实际上是调用了它的`apply`方法)。函数同样可以作为参数传递给其他方法。让我们向`FunctionLiterals`类中添加如下代码：

```scala
def runOperation(f: (Int, Int) => Int, a: Int, b:Int) = {
  f(a, b)
}
```

我们可以传递需要的函数到`runOperation`，像下面这样：

```scala
obj.runOperation(obj.sum, 10, 20)
obj.runOperation(Math.max, 10, 20)
```

### 没有语法糖的函数

上个例子中我们只是使用了一些语法糖。为了能够理解实际上发生了什么，我们将会展示函数的字面被转换成了什么。他们基本上表示为扩展`FunctionN`特质，这里 N 是函数参数的数量。函数字面量的实现会通过`apply`方法被调用(一旦一个类或对象拥有`apply`方法，则可以通过一对小括号来调用它并传入对应需要的参数)。让我们看一下等同于上个例子的实现：

```scala
class SumFunction extends Function2[Int, Int, Int] {
  override def apply(v1:Int, v2:Int): Int = v1 + v2
}

class FunctionObjects {
  val sum = new SumFunction
  
  def runOperation(f:(Int, Int) => Int, a: Int, b: Int): Int = f(a, b)
}

object FunctionObjects extends App{
  val obj = new FunctionObjects
  println(s"3 + 9 = ${obj.sum(3, 9)}")
  println(s"Calling run operation: ${obj.runOperation(obj.sum, 10, 20)}")
  println(s"Using Math.max: ${obj.runOperation(Math.max, 10, 20)}")
}
```

### 增加的表现力

像你在例子中看到的一样，统一化的类和函数提升了表现力，你可以轻松实现不同的任务，比如回调、惰性参数求值、集中的异常处理等等，而无需编写额外的代码和逻辑。此外，函数可以作为类意味着我们可以扩展它们以提供能多的功能。

