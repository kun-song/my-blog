# 偏函数与部分应用

## 偏函数

偏函数，即 *partial function*，维基百科定义如下：

>[Partial Function](https://en.wikipedia.org/wiki/Partial_function)

>In mathematics, a partial function from X to Y (written as f: X ↛ Y) is a function f: X′ → Y, for some subset X′ of X.  It generalizes the concept of a function f: X → Y by not forcing f to map every element of X to an element of Y (only some subset X′ of X). If X′ = X, then f is called a **total function** and is equivalent to a function. Partial functions are often used when the exact domain, X, is not known.

* 对于函数 `f: X → Y`，一般认为 `f` 可处理全部 X 集合，比如 `def isEven(x: Int): Boolean = x % 2 == 0`，`isEven` 可接受所有 `Int` 类型值
* 对于偏函数 `f: X' → Y`，则认为其只能处理 X 的子集 X'，其作用域变小了。

Scala 中的 `PartialFunction[-A, +B]` 特质代表偏函数，其作用域无需包含全部类型为 `A` 的值，可用 `isDefinedAt` 判断某值是否属于其作用域。注意，即使 `isDefinedAt(x)` 返回 `true`，调用 `apply(x)` 依旧可能抛出异常：

```scala
val f: PartialFunction[Int, Any] = { case _ => 1/0 }
```

将 `isEven` 改写为偏函数，使其仅作用于偶数，若使用奇数调用，则 `isEven` 抛出 `MatchError` 异常：

```scala
val isEven: PartialFunction[Int, String] = {
  case x if x % 2 == 0 ⇒ s"$x is even"
}
```

`PartialFunction[-A, +B]` 与 `scala.Function1` 的区别是，偏函数的调用者可以对其作用域之外的取值做处理，例如：

```scala
val sample = 1 to 10
// Vector(2 is even, 4 is even, 6 is even, 8 is even, 10 is even)
val evenNumbers = sample collect isEven
```
* `collect` 方法利用 `isDefinedAt` 方法决定选取哪些值

同样，可以实现 `isOdd` 函数，使其仅作用于奇数：

```scala
val isOdd: PartialFunction[Int, String] = {
  case x if x % 2 != 0 ⇒ s"%x is odd"
}
// Vector(1 is odd, 3 is odd, 5 is odd, 7 is odd, 9 is odd)
val oddNumbers = sample collect isOdd
```

使用 `PartialFunction` 的 `orElse` 方法可以串联偏函数：

```scala
// Vector(1 is odd, 2 is even, 3 is odd, 4 is even, 5 is odd, 6 is even, 7 is odd, 8 is even, 9 is odd, 10 is even)
sample collect (isEven orElse isOdd)
```

## 部分应用函数

部分应用函数，即 *partially applied function*，在 [Scala Glossary](http://docs.scala-lang.org/glossary) 中解释如下：

>A function that's used in an expression and that misses some of its arguments. For instance, if function `f` has type `Int => Int => Int`，then `f` and `f(1)` are partially applied functions.

即调用时，缺失部分参数的函数。

当然若要满足缺失参数也可调用，则该函数必须是柯里化（currying）函数，[Scala Glossary](http://docs.scala-lang.org/glossary) 对柯里化解析如下：

>A way to write functions with multiple parameter lists. For instance `def f(x: Int)(y: Int)` is a curried function with two parameter lists. A curried function is applied by passing serveral arguments lists, as in: `f(3)(4)`. However, it is also possible to write a *partial application* of a curried function, such as `f(3)`.

其定义中提到 *partial application*，那么什么是 *partial application*，它与 *application* 之间有什么关系吗？先看一下维基百科中的相关解析：

>[Function Application](https://en.wikipedia.org/wiki/Function_application)

>In mathematics, function application is the act of applying a function to an argument from its domain so as to obtain the corresponding value from its range.

>[Partial Application](https://en.wikipedia.org/wiki/Partial_application)

>In computer science, partial application (or partial function application) refers to the process of fixing a number of arguments to a function, producing another function of smaller arity. Partial application is sometimes (incorrectly) called currying, which is a related, but distinct concept.

* *function application* 指将函数应用到参数，以获取对应值的 **过程**，即函数调用的过程；
* *partial function application* 也是一个过程，该过程中，固定函数的一部分参数（可以是 0 个或多个），获取到一个参数列表更小的函数；

对于柯里化函数，有两种调用方式，即 *function application* 和 *partial function application*，其中 *partial function application* 结果即为 `partially applied function`。

## 总结

偏函数是对作用域进一步限制，而部分应用是一种函数调用方式，一般用于柯里化函数，二者是雷锋与雷峰塔的关系。
