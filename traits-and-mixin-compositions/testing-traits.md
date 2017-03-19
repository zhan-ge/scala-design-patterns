# 测试特质

测试实际上是软件开发中很重要的一部分。它确保了代码中变更的部分不再产生错误，无论是方法的改变还是其他部分。

我们可以使用多种不同的测试框架，这完全取决于个人喜好。本书中我们将使用 ScalaTest，这也是我在项目中使用的框架；它很容易理解，可读且易于使用。

有些情况下，如果一个特质被混入到了类中，我们则可以直接测试这个类。然而，我们或许仅需要测试一个指定的特质。测试一个没有任何方法实现的特质也没有什么意义，因此这么我会探讨那些拥有代码实现的特质。同时，我们这里展示的单元测实际上是很简单的，他们也仅作为示例。我们会在本书的后续章节讨论更加复杂和有意义的测试。

## 使用一个类

让我们看一下前面见到的`DoubledMultiplierIdentity`将会被如何测试。一种方式是将这个特质混入一个测试类来测试它的方法：

```scala
class DoubledMultiplierIdentityTest extends Flatspec with ShouldMatchers with DoubledMultiplierIdentity
```

然而这会编译失败并显示一个错误信息：

```scala
Error:(5, 79) illegal inheritance; superclass FlatSpec 
  is not a subclass of the superclass MultiplierIdentity 
  of the mixin trait DoubledMultiplierIdentity 
class DoubledMultiplierIdentityTest extends FlatSpec with ShouldMatchers 
with DoubledMultiplierIdentity { 
^
```

我们在前面已经谈论过这个问题，事实上特质只能被混入到一个与该特质拥有相同基类的类。这意味着为了测试这个特质，我们需要在我们的测试类中创建一个虚拟的类然后再使用它：

```scala
package com.ivan.nikolov.linearization

import org.scalatest.{ ShouldMatchers, FlatSpec}

class DoubledMultiplierIdentityTest extends FlatSpec with ShouldMatchers {
  
  class DoubledMultipliersIdentityClass extends DoubledMultiplierIdentity
  
  val instance = new DoubledMultiplierIdentityClass
  
  "identity" should "return 2 * 1" in {
    instance.identity should equals(2)
  }
}
```

## 混入特质

我们可以将特质混入来对他进行测试。有几个地方我们可以这么做：混入到一个测试类或者一个单独的测试用例。

### 混入到测试类

只有当该特质确定没有扩展任何其他类时，才可以将该特质混入到一个测试类，因此特质、测试类的超类必须是相同的。除了这样，其他任何方式都和我们前面做的一样。

让我们测试一个本节出现过的特质 A，他拥有一个`hello`方法。同时我们添加了一个`pass`方法，现在该特质看起来会像下面这样：

```scala
trait A {
  def hello(): String = "Hello, I am trait A!"
  def pass(a: Int): String = s"Trait A said: 'You passed $a.'"
}
```

然后是测试类：

```scala
package com.ivan.nikolov.composition

import org.scalatest.{ShouldMatchers, FlatSpec}

class TraitTest extends FlatSpec with ShouldMatchers with A {
  "hello" should "greet properly." in {
     hello() should equal("Hello, I am trait A！")
   }
   
  "pass" should "return the right string with the number." in {
	  pass(10) should equal("Trait A said: 'You passed 10.'") 
   }
  
  it should "be correct also for negative values." in {
    pass(-10) should equal("Trait A said: 'You passed -10.'") 
  }
}
```

### 混入到测试案例

我们同样可以将特质分别混入到个别测试案例中。这样可以支持我们单独为这些测试用例设置自定义。下面是对上面那个单元测试的另一种表示：

```scala
package com.ivan.nikolov.composition

import org.scalatest.{ ShouldMatchers, FlatSpec }

class TraitCaseScopeTest extends FlatSpec with ShouldMatchers {
  "hello" should "greet properly." in new  A {
    hello() should equal("hello, I am trait A!")
  }
  "pass" should "return the right string with the number." in new A {
    pass(10) should equal("Trait A said: 'You passed 10.'")
  } 
  it should "be correct also for negative values." in new A {
    pass(-10) should equal("Trait A said: 'You passed -10.'")
  }
}
```

在上面的代码中你可以看到，这些测试用例与前面的例子一样。但是是在单独的用例中混入 A。这支持我们对不同的用例设置自定义，比如一个特质需要一个方法的实现或者一个变量的初始化。这种方式也可以让我们仅专注于要测试的特质，而不需要创建它的一些实际的实例。

## 运行测试

在测试编写完成后，运行并观察一切是否符合预期是很有用的。仅需要在项目的根目录执行以下命令将会运行所有测试：`mvn clean test`。

如果你需要，可以将 Maven 项目转换为 SBT 项目，然后通过`sbt test`来触发所有测试。