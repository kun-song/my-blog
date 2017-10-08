# Stream & 惰性求值

## Scala 求值策略

[求值策略](https://zh.wikipedia.org/wiki/%E6%B1%82%E5%80%BC%E7%AD%96%E7%95%A5) 在维基百科中定义如下：

>求值策略定义{% em %}何时{% endem %}和以{% em %}何种次序{% endem %}求值给函数的 {% em %}实际参数{% endem %}，什么时候把它们代换入函数，和代换以何种形式发生。

Scala 提供两种求值策略，分别为：

* 严格求值（默认）
* 非严格求值

### 严格求值

默认情况下，Scala 使用严格求值（更具体为 Call by value），即在计算函数调用时，首先按顺序计算所有 **实际参数** 的值，若实际参数表达式包含死循环，此时整个函数调用无法获得结果。

>Call by value

>实际参数计算顺序由编程语言定义，例如 Common Lisp Eiffel Java 采用 **从左至右** 顺序，而 OCmal 采用 **由右至左** 顺序，而 Scheme C 并未规定顺序。

### 非严格求值

Scala 也允许使用非严格求值（更具体为 Call by name），通过在参数类型前面添加 `=>` 符号，可指定该参数使用非严格求值策略：

```scala
def constOne(x: Int, y: => Int) = 1
```

其优点为，若参数在函数体中没有用到，则不会对其求值；缺点为，若参数在函数中使用多次，则每次都需要重新计算，从而造成效率低下，实践中一般都和 [Thunk](https://en.wikipedia.org/wiki/Thunk) 机制配合使用（关于 thunk 机制的介绍，可以参考 Cousera 的 [Programming Languages Part B](https://www.coursera.org/learn/programming-languages-part-b) 公开课）。

Scala 提供了 `lazy` 关键词，可以避免手工实现 thunk，避免 Call by name 的效率问题。

考虑如下代码片段：

```scala
def maybeTwice(a: Boolean, b: ⇒ Int) =
  if (a) b + b else 0
maybeTwice(true, { println("b"); 2 })
```

其中 `b` 为 call-by-name，上述代码将打印两次 `b`，因为 `b + b` 引用了两次 `b`。使用 `lazy` 可以避免该类重复计算：

```scala
def maybeTwice(a: Boolean, b: ⇒ Int) = {
  lazy val x = b
  if (a) x + x else 0
}
maybeTwice(true, { println("b"); 2 })
```

`lazy` 有两个效果：

1. 延迟对 `lazy` 修饰变量的求值，直到其第一次被引用：
    * 不会立刻计算 `x` 的值，直到 `x + x` 表达式引用了 `x` 变量
2. 缓存
    * 计算 `x` 后，会缓存其值，以后再次引用时，直接读取缓存值，不会再次计算

## Stream 应用

可以使用 `Stream` 定义一个无穷整数序列：

```scala
def from(n: Int): Stream[Int] = n #:: from(n + 1)
```

* `#::` 对应于 `List` 的 `::`，与 `Stream.Cons` 等价；

利用 `from` 可轻易定义自然数：`val nats = from(1)`；还可以定义所有 4 的倍数组成的序列：`val m4s = from(1) map (_ * 4)`。因为 `Stream` 是惰性的，如果想要查看 `m4s` 的内容，可以使用 `(m4s take 100) toList` 以查看其前 100 个元素。

### Sieve of Eratosthenes 算法

[Sieve of Eratosthenes 算法](https://zh.wikipedia.org/wiki/%E5%9F%83%E6%8B%89%E6%89%98%E6%96%AF%E7%89%B9%E5%B0%BC%E7%AD%9B%E6%B3%95) 是一种古老的素数计算方法，原理为从 2 开始，将每个素数的各个倍数，标记为合数。使用 `Stream` 可以将该算法实现如下：

```scala
def sieve(s: Stream[Int]): Stream[Int] =
  s.head #:: sieve(s.tail filter (_ % s.head != 0))
```

其中 `s.tail filter (_ % s.head != 0)` 并不会立即计算，而是当该值第一次被引用时才计算，要实际计算前 1000 个素数，可以：

```scala
sieve(from(2)) take(1000)
```

`take(1000)` 会引用 `sieve` 的前 1000 个元素，导致前 1000 个元素的计算，但之后的其他元素并不会被计算。

### 平方根

可以将牛顿法改写为基于 `Stream` 的实现：

```scala
def sqrtStream(n: Double): Double = {
  def improve(guess: Double) = (guess + n / guess) / 2
  lazy val guesses: Stream[Double] = 1 #:: (guesses map improve)
  guesses
}
```

`sqrtStream` 返回值为包含从 1 开始的，所有逼近 `n` 的平方根的无限流，比如计算 9 的平方根：

```scala
(sqrtStream(9) take 10) toList
```

其结果为：`List(1.0, 5.0, 3.4, 3.023529411764706, 3.00009155413138, 3.000000001396984, 3.0, 3.0, 3.0, 3.0)`，前面的元素都在慢慢逼近真实的结果，而最后是无数个 3 组成的流，实际应用中，我们只需要保证其达到一定精度即可使用。

首先定义 `isGoodEnough` 方法：

```scala
def isGoodEnough(guess: Double, n: Double, tolerance: Double = 0.0001) =
  Math.abs(guess * guess - n) < tolerance
```

计算平方根时，使用 `isGoodEnough` 对流进行过滤，过滤结果为满足精度的全部平方根组成的流，只需取第一个即可：

```scala
(sqrtStream(9).filter(isGoodEnough(_, 9)) take 1) toList
```
