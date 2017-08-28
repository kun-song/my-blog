# 第六周：集合

## 更多结构

### `Vector`

`List` 是线性结构，**随机访问** 复杂度为 `O(n)`，对此 Scala 提供了 `Vector`，提供更快速的随机访问。

`List` 与 `Vector` 大部分函数都相同，但 `Vector` 无 `::` 函数，取而代之的是 `+:` `:+` 分别在头尾添加。

```
x +: xs

xs :+ x
```

### `Seq`

`List` 与 `Vector` 都是 `Seq` 的子类，都具备 `map` `filter` `fold` 等函数，而 `Array` `String` 虽然也具备这些函数，但他们是从 Java 继承来的，所以并非 `Seq` 的子类。

根据需要 `Array` `String` 将被 **隐式转换** 为 `Seq`：

```
"Hello" exists (c => c.isUpper)
Array('a', 'B', 'c') forall (c => c.isUpper)

val p = 1 :: 2 :: 3 :: Nil zip "abc"
```

### `Range`

`Range` 也是 `Seq` 的子类，不过只有 3 中操作，分别为 `until` `to` `by`：

```
val u: Range = 1 until 5
val t = 1 to 5

1 to 10 by 2
10 to 1 by -3
```

### `Seq` 函数

`Seq` 提供的其他函数：

```
val s = "Hello"
val n = Array(1, 2, 30, 4)

s exists (c => c.isUpper)
s forall (c => c.isUpper)

val p = 1 :: 2 :: 3 :: Nil zip "abcdef"
p.unzip

s flatMap (c => 0 :: c :: Nil)

n.sum
n.product
n.max
n.min
```

```
def scalarProduct(xs: Vector[Int], ys: Vector[Int]): Int =
  ((xs zip ys) map (xy => xy._1 * xy._2)).sum

def scalarProd(xs: Vector[Int], ys: Vector[Int]): Int =
  (xs zip ys).map{case (x, y) => x * y}.sum
```

> `{case (x, y) => x * y}` 等价于 `x => x match { case (x, y) => x * y }`，相当于匿名函数的简写。

### 组合子

`xs flatMap = (xs map f).flatten`

* 先 `map` 后 `flatten`

### `for` 表达式

有两种形式的 `for` 表达式，一个用 `()` 一个用 `{}`，其中 `{}` 可以分在多行写：

```
for ( s ) yield e

for {
  s1
  s2
  ...
} yield e
```

* `s` 为 `generator` `filter` 集合
  + `generator` 形式为 `p <- e`，其中 `p` 为模式，`e` 为表达式，其计算结果为集合；
  + `filter` 形式为 `if f` 其中 `f` 为布尔表达式；
* `s` 必须以 `generator` 开头

例子：

```
def isPrime(x: Int): Boolean = (2 until x) forall (d => x % d != 0)

def f(n: Int) =
  (1 until n) flatMap (i => (1 until i) map (j => (i, j))) filter { case (i, j) => isPrime(i + j) }

f(7)

def g(n: Int) =
  for {
    i <- 1 until n
    j <- 1 until i
    if isPrime(i + j)
  } yield (i, j)

g(7)
```

```
def scalarProduct(xs: Vector[Int], ys: Vector[Int]): Int =
  ((xs zip ys) map (xy => xy._1 * xy._2)).sum

def scalarProd(xs: Vector[Int], ys: Vector[Int]): Int =
  (xs zip ys).map{case (x, y) => x * y}.sum

def scalarP(xs: Vector[Int], ys: Vector[Int]) =
  (for {
    (x, y) <- xs zip ys
  } yield x * y).sum
```

### `Set`

`Set` 与 `Seq` 非常相似，也可以使用 `map` `filter` 等函数，但有如下不同点：

1. `Set` 是无序的；
2. `Set` 不能有重复元素；
3. `Set` 基本操作：`Set(1, 2, 3) constains 2`；（`Seq` 的基本操作为 `head`）

### `Map`

* `Map[Key, Value]` 继承集合类型 `Iterable[Key, Value]`，是可遍历的数据结构；
* `Map[Key, Value]` 继承函数类型 `Key -> Value`，是函数，可以用在任何函数可以出现的地方；

```
// 集合
val countries = Map("China" -> "Beijing", "America" -> "WC")

// 函数
countries("England")
```

* 直接用 `countries()` 函数查询，若不存在，则抛出异常，使用 `get` 方法替代：`countries get "England"`，若不存在，则返回 `None`；

#### `orderBy` `groupBy`

`orderBy` `groupBy` 是 SQL 查询语言，其中 `orderBy` 语义可以用 `sortWith` `sorted` 实现：

```
val fruits = List("apple", "pear", "orange")

fruits sortWith (_.length < _.length)
fruits sorted
```
* `sortWith` 可以用自定义的排序规则；
* `sorted` 使用模式的字典序；

Scala 内置了 `groupBy` 语句：

```
val fruits = List("apple", "pear", "orange", "pineapple")

// Map(p -> List(pear, pineapple), a -> List(apple), o -> List(orange))
fruits groupBy (_.head)
```
* 返回一个 `Map[List[...]]`
