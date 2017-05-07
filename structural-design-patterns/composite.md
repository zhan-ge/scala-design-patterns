---
typora-copy-images-to: ../assets
---

# 组合模式

组合设计模式用于描述一组对象应该像一个对象一样处理。

> 目的在于将对象组合为一个树结构以表示 整体-部分 的层级关系。

组合模式可用于移除重复代码或者在一组对象应该以相同的方式处理时避免错误。一个典型的例子是文件系统，我们可以拥有文件夹，其中又可以拥有其他文件夹或文件。通常，用于交互文件或文件夹的接口是相同的，因此它们可以很好的适用组合模式。

## 类图

像我们之前提到的，文件系统能够很好的适用组合模式。总的来说，它们是一个树形结构，因此对于我们的例子来说，我们将会展示如何使用组合模式来构建一个树形结构。

![75228E6A-2A0A-47A9-B67F-A91AD19B4F06](/assets/composite.png)

从上面的类图你会发现，Tree 是我们的组合对象。它包含子对象，要么是另一个带有子对象的 Tree 对象，要门仅仅是一个 Leaf 节点。

## 实例

让我们看一下上面类图的代码表示。首先我们需要通过一个特质来定义`Node`接口：

```scala
trait Node{
  def print(prefix:String):Unit
}
```

> print 方法的 prefix 参数用于帮助在打印树结构时拥有更好的可视化。

在我们拥有接口之后，现在可以定义具体的实现：

```scala
class Leaf(data:String) extends Node {
  override def print(prefix:String):Unit = .println(s"${prefix}${data}")
}

class Tree extends Node{
  private val children = ListBuffer.empty[Node]
  
  override def print(prefix: String): Unit = {
    println(s"${prefix}(")
    children.foreach(_.print(s"${prefix}${prefix}"))
    println(s"${prefix})")
  }
  
  def add(child: Node): Unit = children += child
  def remove(): Unit = { 
    if (children.nonEmpty) children.remove(0)
  }
}
```

之后便可以直接使用这些代码。在打印的时候，我们不用关心它具体是叶子还是树。我们的代码将会自动处理这些：

```scala
object CompositeExample extends App{
  val tree = new Tree
  
  tree.add(new Leaf("leaf 1"))
  
  val subtree1 = new Tree 
  subtree1.add(new Leaf("leaf 2"))
  
  val subtree2 = new Tree 
  subtree2.add(new Leaf("leaf 3")) 
  subtree2.add(new Leaf("leaf 4")) 
  subtree1.add(subtree2)
  
  tree.add(subtree1)
  
  val subtree3 = new Tree 
  val subtree4 = new Tree 
  subtree4.add(new Leaf("leaf 5")) 
  subtree4.add(new Leaf("leaf 6"))
  
  subtree3.add(subtree4) 
  tree.add(subtree3)
  tree.print("-")
}
```

代码实际上会对我们的树形结构进行深度优先遍历。我们拥有的数据结构看起来会像下面这样：

![2277A18D-CF9C-47A8-8233-2908C6F2544E](/assets/tree.png)

## 优点

组合模式能够有效的减少代码重复，在创建层级结构的时候也很直接。简化的部分来自于客户端并不知道它们实际上正在处理的什么类型的对象。添加新的节点类型也很简单，不需要改变其他任何东西。

## 缺点

组合模式没有任何主要的缺点。它确实适用于具体场景。开发者需要注意的一点是在处理大规模的层级结构时，因为在这种情况下，我们可能会深入到递归签到的项目里，而这可能会引起栈溢出问题。

