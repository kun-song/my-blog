# 第五周：列表

## 实现基本 List 操作

```
def last[T](xs: List[T]): T = xs match {
  case Nil => throw new Error("empty list")
  case List(x) => x
  case _ :: xs => last(xs)
}

def init[T](xs: List[T]): List[T] = xs match {
  case Nil => throw new Error("init of empty list")
  case _ :: Nil => Nil
  case x :: xs => x :: init(xs)
}

def concat[T](xs: List[T], ys: List[T]): List[T] = xs match {
  case Nil => ys
  case x :: xs => x :: concat(xs, ys)
}

def reverse[T](xs: List[T]): List[T] = xs match {
  case Nil => Nil
  case x :: xs => reverse(xs) ::: List(x)
}

def removeAt[T](n: Int, xs: List[T]): List[T] = (xs take n) ::: (xs drop n + 1)
```

## Pair/Tuple 模式匹配

```
def sort(xs: List[Int]): List[Int] = {
  val n = xs.length / 2
  if (n == 0) xs
  else {
    /**
      * pair/tuple 模式匹配
      */
    def merge(xs: List[Int], ys: List[Int]): List[Int] = (xs, ys) match {
      case (Nil, ys) => ys
      case (xs, Nil) => xs
      case (x :: xs1, y :: ys1) => if (x < y) x :: merge(xs1, ys) else y :: merge(xs, ys1)
    }

    val (first, second) = xs splitAt n
    merge(sort(first), sort(second))
  }
}
```

## 隐式参数

目前 `sort` 函数只能用于 `List[Int]`，如果想要将其泛化，显而易见需要添加类型参数，但是添加类型参数之后，就不能使用 `<` 比较大小了，所以将 `<` 提取为函数参数：

```
def sort[T](xs: List[T])(lt: (T, T) => Boolean): List[T] = {
  val n = xs.length / 2
  if (n == 0) xs
  else {
    /**
      * pair/tuple 模式匹配
      */
    def merge(xs: List[T], ys: List[T]): List[T] = (xs, ys) match {
      case (Nil, ys) => ys
      case (xs, Nil) => xs
      case (x :: xs1, y :: ys1) => if (lt(x, y)) x :: merge(xs1, ys) else y :: merge(xs, ys1)
    }

    val (first, second) = xs splitAt n
    merge(sort(first)(lt), sort(second)(lt))
  }
}

val l = List(1, 22, 3, 6)
sort(l)((x, y) => x < y)
```

> 因为 `lt` 函数在 `xs` 参数之后定义，所以调用时，Scala type checker 可以推断出 `lt` 参数的类型。

### Ordering

Scala 预定义了表示顺序的类型：`Ordering`，可以直接使用：

```
def sort[T](xs: List[T])(order: Ordering[T]): List[T] = {
  val n = xs.length / 2
  if (n == 0) xs
  else {
    /**
      * pair/tuple 模式匹配
      */
    def merge(xs: List[T], ys: List[T]): List[T] = (xs, ys) match {
      case (Nil, ys) => ys
      case (xs, Nil) => xs
      case (x :: xs1, y :: ys1) => if (order.lt(x, y)) x :: merge(xs1, ys) else y :: merge(xs, ys1)
    }

    val (first, second) = xs splitAt n
    merge(sort(first)(order), sort(second)(order))
  }
}

val l = List(1, 22, 3, 6)
sort(l)(Ordering.Int)

val name = List("Kyle", "Cuoco")
sort(name)(Ordering.String)
```
* `Ordering` 中有预定义的针对 `Int` `String` 等的排序规则，直接使用即可；

### `implicit` 参数

`sort` 函数的 `Ordering` 参数，每次调用都需要写，很繁琐，将其定义为 `implicit` 参数，Scala 编译器能够在合适的地方自动传入：

```
def sort[T](xs: List[T])(implicit order: Ordering[T]): List[T] = {
  val n = xs.length / 2
  if (n == 0) xs
  else {
    /**
      * pair/tuple 模式匹配
      */
    def merge(xs: List[T], ys: List[T]): List[T] = (xs, ys) match {
      case (Nil, ys) => ys
      case (xs, Nil) => xs
      case (x :: xs1, y :: ys1) => if (order.lt(x, y)) x :: merge(xs1, ys) else y :: merge(xs, ys1)
    }

    val (first, second) = xs splitAt n
    // 1. 简化
    merge(sort(first), sort(second))
  }
}

val l = List(1, 22, 3, 6)
// 2. 简化
sort(l)

val name = List("Kyle", "Cuoco")
sort(name)
```

将 `Ordering` 参数定义为隐式参数之后，有两处简化，都不再需要显示传入 `Ordering` 参数。

## List 高阶函数

### `map` 函数

```
def squareList(xs: List[Int]): List[Int] = xs match {
  case Nil => Nil
  case x :: xs => x * x :: squareList(xs)
}
squareList(List(1, 2, 3))

List(1, 2, 3) map (x => x * x)
```

### `filter` 函数

类似函数有 2 对：

```
val ints = List(1, 22, -3, 6, -4)

ints filter (x => x > 0)      // List(1, 22, 6)
ints filterNot (x => x > 0)   // List(-3, -4)
ints partition (x => x > 0)   // (List(1, 22, 6),List(-3, -4))

ints takeWhile (x => x > 0)   // List(1, 22)
ints dropWhile (x => x > 0)   // List(-3, 6, -4)
ints span (x => x > 0)        // (List(1, 22),List(-3, 6, -4))
```

### `reduce` 函数

`reduceLeft` 函数是 Scala 中的普通 `reduce`，用法如下：

```
def sum(xs: List[Int]): Int = xs reduceLeft ((x, y) => x + y)
sum(List(1, 2, 3))

def prod(xs: List[Int]): Int = xs reduceLeft((x, y) => x * y)
prod(List(1, 2, 3))
```

匿名函数可以简写：

```
def sum(xs: List[Int]): Int = xs reduceLeft (_ + _)
sum(List(1, 2, 3))

def prod(xs: List[Int]): Int = xs reduceLeft(_ * _)
prod(List(1, 2, 3))
```

以上定义的 `sum` `prod` 在列表为空时，将抛出异常，使用 `foldLeft` 替代 `reduceLeft`，并提供一个 `accumulator` 作为参数，当列表为空时，将返回 `accumulator`：

```
def sum(xs: List[Int]): Int = (xs foldLeft 0) (_ + _)
sum(List(1, 2, 3))

def prod(xs: List[Int]): Int = (xs foldLeft 1) (_ * _)
prod(List(1, 2, 3))
// 1
prod(Nil)
```

与 `reduceLeft` `foldLeft` 对应，Scala 有 `reduceRight` `foldRight`，区别是 `op` 操作的 **结合性** 不同，一个从左往右，一个从右往左：

```
List(x1, ..., xn) reduceLeft op = (...(x1 op x2) op ... ) op xn
(List(x1, ..., xn) foldLeft z)(op) = (...(z op x1) op ... ) op xn

List(x1, ..., x{n-1}, xn) reduceRight op = x1 op ( ... (x{n-1} op xn) ... )
(List(x1, ..., xn) foldRight acc)(op) = x1 op ( ... (xn op acc) ... )
```

> `fold` 函数会提供一个初始值，第一次聚合是 `accumulator op x1`，而 `reduce` 函数第一次是 `x1 * x2`。

对于满足 **结合律** + **交换律** 的 `op` 来讲，`foldLeft` `foldRight` 并无差别，但对于某些 `op`，可能只能使用其中之一：

```
def concat[T](xs: List[T], ys: List[T]): List[T] = (xs foldRight ys) (_ :: _)
```

若将其替换为 `foldLeft`，则计算展开为：`ys :: x1 ... :: xn`，`::` 使用错误，而 `foldRight` 展开为：`x1 :: ... xn :: ys`，这时正确的。
