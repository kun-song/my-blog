# 第四周：类型与模式匹配

## Primitive Values -> Objects

在 Scala 中，所有原始类型都是类，这与 Java 区别很大。

## Function Values -> Objects

In Scala, `function values` are treated as `objects`.

### 函数类型

函数类型 `A => B` 是类 `scala.Function1[A, B]` 的简写，该类定义如下：

```
package scala

trait Function1[A, B] {
  def apply(x: A): B
}
```

> **函数即为具备 `apply` 方法的对象**

Scala 对于多参函数，有对应的特质：`Function2` `Function3` ... `Function22`，目前最多支持 22 个参数的函数。

### 函数值（只有匿名函数才是 function value）

现在函数字面值如下：

```
(x: Int) => x * x
```

Scala 将其转化为：

```
{
  class AnonFun extends Function1[Int, Int] {
    def apply(x: Int) = x * x
  }
  new AnonFun
}
```

或者使用匿名内部类简化：

```
new Function1[Int, Int] {
  def apply(x: Int) = x * x
}
```

### 函数调用

函数调用 `f(a, b)` 会被展开为：

```
f.apply(a, b)
```

因此对于如下函数调用：

```
val f = (x: Int) => x * x
f(7)
```

将被展开为：

```
val f = new Function1[Int, Int] {
  def apply(x: Int) = x * x
}
f.apply(7)
```

### 函数 vs 方法

注意如下方法定义：

```
def f(x: Int): Boolean = ...
```

该方法并非 `function value`，但若 `f` 用于期望函数值的地方，则其 **自动转化为 function value**：

```
(x: Int) => f(x)
```

或者：

```
new Function1[Int, Boolean] {
  def apply(x: Int) = f(x)
}
```

## Subtyping and Generics

有两种多态形式：

1. Subtyping：将子类对象传递给父类引用
2. Generics：类型参数

本节讨论二者的边界、差异。

### Type Bound

* upper bound:
  * S <: T means: S is a subtype of T, and
* lower bound:
  * S >: T means: S is a supertype of T, or T is a subtype of S.

```
def assertAllPos[S <: IntSet](r: S): S = ...
```
* S 只能是 `IntSet` 的子类，即 `Empty` `NonEmpty`；

```
[S >: NonEmpty]
```
* S 只能是 `NonEmpty`, `IntSet`, `AnyRef`, `Any` 之一；

可以混合使用 upper bound/lower bound：

```
[S >: NonEmpty <: IntSet]
```
* S 只能是 `NonEmpty` `IntSet`；

## List

List 上的所有函数都可以用下面三个函数实现：

1. head
2. tail
3. isEmpty
