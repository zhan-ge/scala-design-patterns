# 其他工厂设计模式

还有一些工厂设计模式的变种。而在所有的场景中，它们的目的都是相同的——隐藏创建过程的复杂性。在下个小节中，我们将简短的提到另外两种工厂模式：静态工厂、简单工厂。

## 静态工厂

静态工厂可以被表示为一个静态方法，并作为基类的一部分。调用它来创建那些具体类的实例。这里最大的缺点之一是，如果给基类添加了另外一个扩展，基类也需要给修改，即修改这个静态方法。让我们展示一个动物世界中的例子：

```scala
trait Animal
class Bird extends Animal
class Mammal extends Ainmal
class Fish extends Animal

object Animal {
  def apply(animal:String):Animal = animal match {
    case "bird" => Bird
    case "mammal" => Mammal
    case "fish" => Fish
    case x:String => throw new RuntimeException(s"Unknown animal: $x")
  }
}
```

这样每次添加一种新的具体动物，我们不得不修改这个`apply`方法来包括它，特别是当我们要考虑新的类型时。

## 简单工厂

简单工厂比静态工厂要好一点，因为实际的工厂功能是在另一个类中。这使得每次扩展基类不必再去修改基类。类似于抽象工厂，但不同在于这里我们没有一个抽象工厂类，会直接使用具体工厂。通常从一个简单的工厂开始，随着时间的推移和项目的进化，最终会演变成一个抽象工厂。

## 工厂组合

当然，可以将不同的工厂模式组合在一起。当然，这种方式需要谨慎选择并且仅在有必要的时候使用。否则，设计模式的滥用将会导致烂代码。

