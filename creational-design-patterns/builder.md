---
typora-copy-images-to: ../assets
---

# 构建器模式

构建起模式支持以类方法而类构造器的方式来创建实例。当一个类的构造器拥有多个版本以支持不同的用途时，这种模式尤其有用。更进一步，有些情况下，创建所有的组合是不可能的或者它们也无法被了解。构建起模式使用一个额外的对象，称为`builder`，用于在创建最终版本的对象之前接收和保存初始化参数。

## 类图

这个小节中，我们首先会提供一个在其他语言中看起来比较经典的类图，包括 Java。然后，我们会基于它们来提供不同版本的代码实现来使其更符合 Scala，以及一些围绕它们的观察和讨论。

我们会拥有一个`Person`类，带有参数：`firstName`,`lastName`,`age`,`departmentId`等等。下个小节中会展示实际的代码。创建一个具体的构造器可能会花费太长时间，尤其是有些参数有些时候可能并不需要或被了解。这也会让代码在以后变得难以维护。而构建器模式可能会是个不错的想法，它的来图看起来像下面这样：

![BB9077C8-F300-418B-88C8-90613DC8BB48](/assets/builder.png)

我们已经提到过，这也就是构建器模式在纯面向对象语言中看起来的样子。当构建器是抽象的时候，表示也会有所不同，这时会存在一些具体的构建器。这对它所创建的产品来说也是一样。最终，它们的目标都一样——使对象的创建更简单。

## 实例

实际上在 Scala 中有三种不同的方式来表示构建器模式：

- 经典方式，像上面展示的类图，类似与其他编程语言。这种方式实际上是不推荐的，尽管在 Scala 中也能实现。为了实现它使用了可变性，这违背了语言的可不变原则。为了完整性和体现使用 Scala 的简单特性会使实现多么简单，我们会进行展示。
- 使用带有默认参数的的样例类。我们会看到两种版本的实现，一种验证参数而另一种则没有。
- 使用泛化的(generalized)类型约束。

下面的几个小节我们将会关注这些方式。为了使事情变得简单便于演示，我们会在类中使用较少的字段；然而，需要注意的是构建器模式真正适用的是拥有大量字段的类。

## 类似 Java 方式的实现

首先是 Person 类：

```scala
class Person(builder: PersonBuilder){
  val firstName = builder.firstName
  val lastName = builder.lastName
  val age = builder.age
}
```

该类接收一个 builder 并使用它被设置的值来初始化字段。下面是 Builder 的实现：

```scala
class PersonBuilder {
  var firstName = ""
  var lastName = ""
  var age = 0
  
  def setFirstName(firstName:String):PersonBuilder = {
    this.firstName = firstName
    this
  }
  
  def setLastName(lastName:String):PersonBuilder = {
    this.lastName = lastName
    this
  }
  
  def setAge(age:Int):PersonBuilder = {
    this.age = age
    this
  }
}
```

构建器中提供了方法用于设置与`Person`向对应的字段值。这些方法都会返回相同的构建器实例，这样就能支持我们链式的调用它们。下面是如何使用这个构建器：

```scala
object PersonBuilderExample extends App{
  val person:Person = new PersonBuilder()
    .setFirstName("Ivan")
    .setLastName("Nikolov")
    .setAge(26)
    .build()
  System.out.println(s"Person: ${person.firstName} ${person.lastName}. Age: ${person.age}.")
}
```

这就是构建器模式的使用方式。现在我们就可以创建一个`Person`对象而无论是否拥有需要提供的值——甚至仅拥有需要字段的一部分，我们可以指定一些字段然后剩余的字段会拥有默认的值。当为`Person`类添加新的字段也不必再创建新的构造器。仅需要通过`PersonBuilder`类进行兼容即可。

## 使用样例类实现

上个构建器模式的例子看起来很好，但是需要编写一些额外的代码和创建模板。此外，它需要我们在`PersonBuilder`类中拥有，这也违背了 Scala 中的不可变原则。

Scala 拥有样例类，这使得构建器模式的实现更加简单：

```scala
case class Person(
  firstName:String = "",
  lastName:String = "",
  age:Int = 0
)
```

其用法也与之前构建器类似：

```scala
object PersonCaseClassExample extends App {
  val person1 = Person(
    firstName = "Ivan",
    lastName = "Nikolov",
    age = 0
  )
  
  val person2 = Person(firstName = "John")
  
  System.out.println(s"Person 1: ${person1}") 
  System.out.println(s"Person 2: ${person2}")
}
```

这种实现要比前面第一种实现更加简短也更易维护。它能为开发者提供与第一种实现完全相同的功能，但是更简短、语法更清晰。同时保持了`Person`类的字段不可变，这也遵循了 Scala 中好的实践。

这两种实现都有的缺点是没有对参数进行验证。如果一些组件相互依赖并且有些特定的参数需要验证呢？前面这两种方式的用例中，我们可能会得到一些运行时异常。下一小节中将会展示如何确保验证和需求得到满足被实现。

## 使用泛化类型约束

在软件工程中的一些创建对象的用例中，我们会拥有一些依赖。我们要需要一些东西已经被初始化完成，那么需要一个特定的初始化顺序，等等。前面我们看到的两种构建器模式实现都缺乏确保某些东西已被实现或未被实现的能力。在这种方式中，我们需要围绕构建器模式创建一些额外的验证，以确保一切都符合预期。当然，我们会看一下在运行时创建对象是否是安全的。

使用一些我们之前在本书中见到的技术，我们可以创建一个能够在运行时验证所有的需求是否都已被满足的构建器。这杯称为**type-safe builder**，下个小节中我们会展示这种模式。

### 改变 Person 类

首先，我们将使用与 Java 实现方式中展示的例子中相同的类开始。现在添加一些约束，比如每个人都最少拥有`firstName`和`lastName`。为了能够让编译器感知到这些字段是否已被设置，需要将这些编码为一个类型。我们将使用 ADT 来达到这个目的。让我们定义下面的代码：

```scala
sealed trait BuildStep
sealed trait HasFirstStep extends BuildStep
sealed trait HasLastStep extends BuildStep
```

上面这些抽象类型定义了构建过程的不同步骤。现在对之前的`Person`类个构建器进行一些重构：

```scala
class Person(val firstName:String, val lastName:String, val age:Int)
```

我们将会使用`Person`类完整的构造器，而非传递一个构建器。这是为了展示构建实例并保持后续的步骤代码简介的另一种方式。这些改变需要`PersonBuilder`中的`build`方法也进行一些改变：

```scala
def build():Person = new Person(firstName, lastName, age)
```

### 改变 PersonBuilder 类

现在改变`PersonBuilder`的声明为如下方式：

```scala
class PersonBuilder[PassedStep <: BuildStep] private (
  var firstName:String,
  var lastName:String,
  var age:Int
)
```

这将要求之前所有那些返回`PersonBuilder`的方法现在返回`PersonBuilder[PassedStep]`。同时，这要限制不能够在使用`new`关键字来创建构建器了，因为构造器现在是私有的。现在添加一些构造器重载：

```scala
protected def this() = this("","",0)
protected def this(pb: PersonBuilder[_]) = this(
  pb.firstName,
  pb.lastName,
  pb.age
)
```

后面我们将会看到如何使用这些构造器。我们会支持用户以另一个方法来创建构建器，因为所有的构造器对外部都是不可见的。因此我们要添加一个伴生对象：

```scala
object PersonBuilder{
  def apply() = new PersonBuilder[BuildStep]()
}
```

伴生对象使用了我们前面定义的其中一个构造器，它同样确保了返回的对象用有正确的构建步骤。

## 给需要的方法添加泛化类型约束

到目前为止，我们所有拥有的仍然不能满足我们对每个 Person 对象初始化的要求。我们需要改变一些`PersonBuilder`类中的方法。下面是这些方法的定义：

```scala
def setFirstName(firstName:String):PersonBuilder[HasFirstName] = {
  this.firstName = firstName
  new PersonBuilder[HasFirstName](this)
}

def setLastName(lastName:String):PersonBuilder[HasLastName] = {
  this.lastName = lastName
  new PersonBuilder[HasLastName](this)
}
```

有趣的部分是最后的`build`方法，让我们首先看一下最初的额实现：

```scala
def build()(implicit ev:PassedStep =:= HasLastName):Person = 
  new Person(firstName, lastName, age)
```

上面的语法设置了一个泛化类型约束，它要求仅能在已经通过了`HasLastName`步骤的构建器上调用。看起来我们已经接近了预期的实现，但是现在的`build`方法仅适用于`setLastName`是四个方法中最后一个被调用的时候，同时也不会验证其他字段。让我们为`setFirstName`和`setLastName`使用类似的方式并将它们链接起来，因此每个方法都会要求上一个方法已经被调用。下面是`PersonBuilder`的最终实现：

```scala
class PersonBuilder[PassedStep <: BuildStep] private (
  var firstName:String,
  var lastName:String,
  var age:Int
){
  protected def this() = this("", "", 0)
  protected def this(pb:PersonBuilder[_]) = this(
    pb.firstName,
    pb.lastName,
    pb.age
  )
  
  def setFirtstName(firstName:String):PersonBuilder[HasFirstName] = {
    this.firstName = firstName
    new PersonBuilder[HasFirstName](this)
  }
  
  def setLastName(lastName:String)(implicit ev:PassedStep =:= HasFirstName):PersonBuilder[HasLastName] = {
    this.lastName = lastName
    new PersonBuilder[HasLastName](this)
  }
  
  def setAge(age:Int):PsersonBuilder[PassedStep] = {
    this.age = age
    this
  }
  
  def build()(implicit ev: PassedStep =:= HasLastStep):Person = 
    new Person(firstName, lastName, age)
}
```

### 使用 type-safe 构建器

```scala
object PersonBuilderTypeSafeExample extends App {
  val person = PersonBuilder()
    .setFirstName("Ivan")
    .setLastName("Nikolov")
    .setAge(26)
    .build()
  System.out.println(s"Person: ${person.firstName} ${person.lastName}. Age: ${person.age}.")
}
```

如果我们遗漏了两个要求的方法之一或者颠倒了顺序，将会得到一个类似下面这样的编译错误：

```
Error:(103, 23) Cannot prove that com.ivan.nikolov.creational.builder.
type_safe.BuildStep =:=
com.ivan.nikolov.creational.builder.type_safe.HasFirstName.
		.setLastName("Nikolov")
		^
```

顺序的要求可以被认为是一个缺点，特别是根本不需要顺序的时候，不过可以通过一些额外的方式来解决这个问题。

> 对这个 type-safe 构建器的一些观察：
>
> - 使用 type-safe 构建器，我们可以限定一个指定的调用顺序，和一些已经被初始化的字段
> - 如果我们限定了多个字段，则需要将它们链接，这使得调用顺序变得重要。有些情况下会使库变得难用
> - 编译器消息，如果构建器没有被正确使用，这些消息并不能明确指示错误
> - 代码看起来跟 Java 方式的实现类似
> - 这种类似 Java 的实现方式导致了对并不推荐的可变性的依赖

Scala 支持我们使用一个很好很清晰的构建器模式实现，同时能够要求构建顺序和已被初始化的字段。这是一个很好的特性，尽管有时会变得冗长乏味，或者限制了方法的实际用途。

## 使用 require 语句

上面展示的 type-safe 构建器很好，不过仍然有一些缺点：

- 复杂性
- 可变性
- 一个预定义的初始化顺序

然而，它有时可能会很有用，因为能够支持在编译期间检查所编写的代码。尽管有些时候编译期间的检查是没有必要的。如果这样的话，我们可以使事情变得非常简单并摆脱之前的复杂性，即使用已知的 case 类和`require`语句：

```scala
case class Person(firstName:String = "", lastName:String = "", age:Int = 0){ 
  require(firstName != "", "First name is required.")
  require(lastName != "", "Last name is required.")
}
```

如果上面的布尔条件不满足，我们的代码将会抛出一个`IllegalArgumentException`异常并包含正确信息。我们可以以同样的方式使用这些 case 类：

```scala
object PersonCaseClassRequireExample extends App{
  val person1 = Person(firstName = "Ivan", lastName = "Nikolov", age = 26)
  println(s"Person 1: ${person1}")
  try{
    val person2 = Person(firstName = "John")
  } catch {
    case e: Throwable => e.printStackTrace()
  }
}
```

我们发现这里变得更加简单了，字段也不再可变，实际上也没有任何特定的初始化顺序。更进一步，我们可以设置有意义的信息来帮助我们诊断潜在的问题。如果编译期间的验证并非必须，这可能是更好的方式。

## 优点

当我们需要创建一个复杂的对象时，这种构建器模式尤其适用，否则需要创建很多构造器。它通过一个 step-by-step 的方式使对象的创建更加简单、清晰、可读。

## 缺点

像我们在 type-safe 构建器中看到的，添加一些高级的逻辑和要求会带来相当多的工作。如果没有这种可能性，开发者会将危险暴露给这些类的用户以出现更多错误。同时，构建器包含很多看起来重复的代码，尤其是以类似 Java 的方式实现。

