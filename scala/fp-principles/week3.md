# 第三周：数据与抽象

考虑实现一个 `int` 集合，首先定义一个抽象类：

```
abstract class IntSet {
  def incl(x: Int): IntSet
  def contains(x: IntSet): IntSet
}
```
* 抽象类无法使用 `new` 创建对象；

## 使用 `tree` 实现 `IntSet` 类

现在只有 `IntSet` 的抽象类，`incl` `contains` 方法都没有定义，那么该如何实现该类呢？一种选择是用 **二叉树** 来实现：

有两种类型的二叉树：

1. 空二叉树
  * 代表空集合
2. 非空二叉树
  * 包含一个整数 + 两棵子树
  * 代表非空集合

```
class Empty extends IntSet {
  override def incl(x: Int): IntSet = new NonEmpty(x, new Empty, new Empty)
  override def contains(x: Int): Boolean = false
}

class NonEmpty(elem: Int, left: IntSet, right: IntSet) extends IntSet {
  override def incl(x: Int): IntSet = {
    if (x < elem) new NonEmpty(elem, left incl x, right)
    else if (x > elem) new NonEmpty(elem, left, right incl x)
    else this
  }

  override def contains(x: Int): Boolean = {
    if (x < elem) left contains x
    else if (x > elem) right contains x
    else true
  }
}
```

使用方法如下：

```
val set = new NonEmpty(11, new Empty, new Empty)
```

## 使用单例替代 `Empty` 类

上面的例子中，其实只需要一个 `Empty` 对象即可，因为它不包含任何值，所以可以将其定义为 **单例对象**（唯一的区别在于将 `class` 替换为 `class`）：

```
object Empty extends IntSet {
  override def incl(x: Int): IntSet = new NonEmpty(x, Empty, Empty)
  override def contains(x: Int): Boolean = false

  override def toString = "."
}
```

使用方法也有变化，因为 `singleton object` 本身就是值，所以不需要使用 `new` 创建：

```
val set = new NonEmpty(11, Empty, Empty)
```

### Scala 引入包

Scala 有三种方式导入包：

```
import week3.Rational           // imports just Rational
import week3.{Rational, Hello}  // imports both Rational and Hello
import week3._                  // imports everything in package week3
```

以下内容是自动导入的，无需手动：

```
* All members of package scala
* All members of package java.lang
* All members of the singleton object scala.Predef.
```

## Trait

Scala 与 Java 相同，都只能有一个 **直接父类**，Java 使用接口解决该问题，而 Scala 使用 `trait` 解决，`trait` 相对 `interface` 的优势如下：

1. `trait` 拥有 fields + concrete methods
2. `interface` 只能拥有 abstract methods

> 注意：`trait` 不能有 parameters，只有 class 才可以有，因为不能创建 `trait` 实例。

## Scala 类结构

Scala 类结构可以在[这里](http://docs.scala-lang.org/tour/unified-types.html)查看。

### `Any` `AnyVal` `AnyRef`

类图的最顶端为 `Any`，之后分别是代表 **值** 的 `AnyVal` 与代表 **引用** 的 `AnyRef`，比如如下代码：

```
if (true) 1 else false
```

其计算结果的类型为 `AnyVal`。

### `Nothing` `Null`

`Nothing` 处于类图的 **最底端**，不存在 `Nothing` 类型的值，该类主要用于提示 **异常终止**，比如抛出异常、死循环等。

`Null` 是所有引用类型的子类，该类型只有一个值：`null`，`Null` 类主要是为了与其他 JVM 语言交互使用，在 Scala 中几乎不使用。

## 多态

```
trait List[T] {
  def isNil: Boolean
  def head: T
  def tail: List[T]
}

class Cons[T](val head: T, val tail: List[T]) extends List[T] {
  override def isNil: Boolean = false

  override def toString: String = head + " " + tail
}

class Nil[T] extends List[T] {
  override def isNil: Boolean = true
  override def head: Nothing = throw new NoSuchElementException("Nil.head")
  override def tail: Nothing = throw new NoSuchElementException("Nil.tail")

  override def toString: String = "."
}
```

不仅类可以有 **类型参数**，函数也可以：

```
def singleton[T](x: T): List[T] = new Cons[T](x, new Nil[T])

val t = singleton[Boolean](true)
val n = singleton[Int](1)
```

得益于 Scala 的类型推断，函数调用时一般无需附带类型参数：

```
val t = singleton(true)
val n = singleton(1)
```

### 多态定义

在编程语言中，多态有两种表现形式：

1. the function can be applied to arguments of **many types**, or
2. the type can have instances of many types

对应的理论语言为：

1. `subtyping`
  * 子类的实例可以传递给父类引用，比如任何接受 `List` 的参数都可以传递一个 `Nil1` `Cons` 实例；
2. `generics`
  * 通过 **类型参数** 创建函数/类的实例；

```
def nth[T](n: Int, xs: List[T]): T = {
  if (xs.isNil) throw new IndexOutOfBoundsException("n = " + n)
  if (n == 0) xs.head
  else nth(n - 1, xs.tail)
}
```
