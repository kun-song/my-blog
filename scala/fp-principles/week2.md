# 第二周：高阶函数

## 匿名函数、高阶函数基本用法

### 匿名函数（语法糖）

Scala 中定义匿名函数如下：

```
(x: Int) => x * x * x
```

与如下方式 **等价**：

```
def f (x: Int) = x * x * x; f
```
* 先定义函数 `f`，然后使用该函数值（function is value）；

因此匿名函数是普通函数的 **语法糖**，任意匿名函数都可以用普通函数替代。

### 函数作为参数

若要对 [a, b] 之间的数字求和：

```
def sum(f: Int => Int, a: Int, b: Int): Int =
  if (a > b) 0
  else f(a) + sum(f, a + 1, b)  // 差异
```

若要对 [a, b] 之间的数字的立方求和：

```
def cubes(x: Int) = x * x * x
def sumCubes(a: Int, b: Int): Int =
  if (a > b) 0
  else cubes(a) + sumCubes(a + 1, b)  // 差异
```

对 [a, b] 之间的数字阶乘求和：

```
def fact(x: Int): Int = if (x == 0) 0 else x * fact(x - 1)
def sumFacts(a: Int, b: Int): Int =
  if (a > b) 0
  else fact(a) + sumCubes(a + 1, b)  // 差异
```

可以发现，这三个函数模式非常类似，差异之处在于求和时方式不同，将差异之处 **抽象成函数**，则有：

```
def sum(f: Int => Int, a: Int, b: Int): Int =
  if (a > b) 0
  else f(a) + sum(f, a + 1, b)
```
第一个参数为函数，既可以传入普通函数，也可以传入匿名函数：

```
def sumInts(a: Int, b: Int) = sum(id, a, b)
def sumCubes(a: Int, b: Int) = sum(cube, a, b)
def sumFacts(a: Int, b: Int) = sum(fact, a, b)

def id(x: Int) = x
def cube(x: Int) = x * x * x
def fact(x: Int): Int = if (x == 0) 1 else x * fact(x - 1)
```

可以看到此处对 `sumInts` `sumCubes` `sumFacts` 的定义简洁了很多，但其中 `id` `cube` 这三个函数，仅仅作为 `sum` 函数的参数存在，则可以使用匿名函数替代：

```
def sumInts(a: Int, b: Int) = sum(x => x, a, b)
def sumCubes(a: Int, b: Int) = sum(x => x * x * x, a, b)
```
* `Fact` 函数是递归的，无法用匿名函数替换；

## 柯里化（语法糖）

前面定义的 `sum` 函数，如果需要传入函数参数，一般写成如下形式：

```
def sumInts(a: Int, b: Int) = sum(x => x, a, b)
def sumCubes(a: Int, b: Int) = sum(x => x * x * x, a, b)
def sumFactorials(a: Int, b: Int) = sum(fact, a, b)
```

显而易见，`sumInts` `sumCubes` `sumFactorials` 三个函数的定义中，参数 `a` `b` 其实并未起到任何作用，它们的存在使得代码更加冗长，使用柯里化可以简化它们。

如下是柯里化版本的 `sum` 函数：

```
def sum(f: Int => Int): (Int, Int) => Int = {
  def sumF(a: Int, b: Int): Int =
    if (a > b) 0
    else f(a) + sumF(a + 1, b)
  // 返回一个本地函数，类型为 (Int, Int) => Int
  sumF
}
```

> 柯里化并无任何神奇之处，仅仅是返回 **函数** 而已。

`sum` 函数接受函数 `f`，且返回一个函数，所以可以这样使用：

```
def sumInts = sum(x => x)
def sumCubes = sum(x => x * x * x)
def sumFactorials = sum(fact)
```
* `sumInts` ... 是 `sum` 函数返回的函数；
* 不再需要冗余的 `a` `b` 参数；

也可以不定义 `sumInts`，而直接使用 `sum` 返回的函数：

```
sum(x => x * x * x)(1, 10)
```

> The definition of functions that return functions is so useful in functional programming that there is a `special syntax` for it in Scala.

```
def sum(f: Int => Int)(a: Int, b: Int): Int =
  if (a > b) 0
  else f(a) + sum(f)(a + 1, b)
```
* 即将各部分参数使用相邻的 `()` 包括即可；
* SML 对应的语法是，将参数的 `()` 去掉，这样每个参数都是柯里化的；

类似，要求 [a, b] 之间数字的作用于某函数之后的乘积，可以写作：

```
def product(f: Int => Int)(a: Int, b: Int): Int =
  if (a > b) 1
  else f(a) * product(f)(a + 1, b)
```

使用 `product` 函数可以定义阶乘：

```
def fact(n: Int) = product(x => x)(1, n)
```

### 抽象 `sum` `product` 为一个函数

仔细查看 `sum` `product` 的定义：

```
def sum(f: Int => Int)(a: Int, b: Int): Int =
  if (a > b) 0
  else f(a) + sum(f)(a + 1, b)

def product(f: Int => Int)(a: Int, b: Int): Int =
  if (a > b) 1
  else f(a) * product(f)(a + 1, b)
```

会发现两者非常相似，差异之处主要有两点：

1. a > b 时的表达式值
2. 如何组合 `f(a)` 与递归结果

将这两部分抽离出来，可以得到如下函数定义，该函数可以定制以上两处差异点：

```
def mapReduce(combine: (Int, Int) => Int, zero: Int)(f: Int => Int)(a: Int, b: Int): Int =
  if (a > b) zero
  else combine(f(a), mapReduce(combine, zero)(f)(a + 1, b))
```

使用 `mapReduce` 函数可以轻易定义 `sum` `product` 函数：

```
def sum(f: Int => Int)(a: Int, b: Int) = mapReduce((x, y) => x + y, 0)(f)(a, b)
def product(f: Int => Int)(a: Int, b: Int) = mapReduce((x, y) => x * y, 1)(f)(a, b)
```

## 例子：Find the fixed point

若对于函数 `f` 有 `f(x) = x`，则称 `x` 为 `f` 的 fixed point。

## 函数与数据

### `require` vs `assert`

`require` 与 `assert` 比较类似，都接受一个 `boolean` 参数，以及一个可选的消息，且在条件不满足时都会抛出异常，但两者有如下区别：

1. 使用场景不同：
  * `require` 用于对 **函数调用者** 强制 **预置条件**；
  * `assert` 用于检查函数本身的代码；
2. 抛出异常不同：
  * `require` 抛出 `IllegalArgumentException`（错误的调用参数）
  * `assert` 抛出 `AssertionError`（代表代码本身逻辑存在错误）

### 构造函数

Scala 中，定义类时隐式引入了一个构造函数，称为该类的 `primary constructor`，该构造函数：

1. **名字** 即为类名
2. **参数** 为类定义时的参数
3. **函数体** 为类中的 *全部语句**

### 定义类

Scala 类与 Java 很类似：

```
class Rational(x: Int, y: Int) {
  val num = x
  val denom = y
}
```

与 Java 不同的是，类定时时同时引入一个构造函数，该函数与类同名，创建该类型的数据：

```
val x = new Rational(2, 3)
```

除了隐式的构造函数，还可以添加其他构造函数：

```
def this(x: Int) = this(x, 1)
```

再添加基本的算术方法，可以得到：

```
class Rational(x: Int, y: Int) {
  require(y != 0, "den can't be 0!")

  def num: Int = x
  def den: Int = y

  def this(x: Int) = this(x, 1)

  def add(that: Rational) =
    new Rational(
      num * that.den + den * that.num,
      den * that.den)

  def neg = new Rational(-num, den)

  def sub(that: Rational): Rational = this.add(that.neg)

  def less(that: Rational): Boolean = num * that.den < den * that.num

  def max(that: Rational): Rational = if (this.less(that)) that else this

  override def toString: String = num + "/" + den
}
```

### 化简

目前得到的 `Rational` 类，不会输出分数的最简形式，比如：

```
// 输出 6/3
val x = new Rational(6, 3)
```

有两种化简方式：

1. 在构造时化简
2. 在打印时化简

但方式 2 存在数字溢出的可能，因为 `Int` 值是有限的，所以最合理的是在创建 `Rational` 时即进行化简：

```
class RationalS(x: Int, y: Int) {
  require(y != 0, "den can't be 0!")

  private def gcd(a: Int, b: Int): Int = if (b == 0) a else gcd(b, a % b)
  private val g = gcd(x, y)

  def num: Int = x / g
  def den: Int = y / g

  def this(x: Int) = this(x, 1)

  def add(that: RationalS) =
    new RationalS(
      num * that.den + den * that.num,
      den * that.den)

  def neg = new RationalS(-num, den)

  def sub(that: RationalS): RationalS = this.add(that.neg)

  def less(that: RationalS): Boolean = num * that.den < den * that.num

  def max(that: RationalS): RationalS = if (this.less(that)) that else this

  override def toString: String = num + "/" + den
}
```
* `gcd` `g` 都只用于类内部化简，所以定义为 `private`

## 操作符

前面定义的 `Rational` 类，与平时我们认识的分数操作有些差异：

```
x + y
x.add(y)
```

Scala 通过两种方式抹平这些差异。

### 中缀操作符

Scala 中，任何 **只有一个参数** 的方法都可以用作中缀操作符，比如 `Rational` 类中的 `add` `sub` `max` 等该方法：

```
s.add(r)  <=>  s add r
s.sub(r)  <=>  s sub r
s.max(r)  <=>  s max r
```

可以看到，相比传统的方法调用方式，中缀更加符合人类习惯。

### 操作符 -> 标识符

Scala 中，操作符可以用作标识符，实际上，Scala 的标识符有两种：

1. **Alphanumeric**: starting with a letter, followed by a sequence of letters or numbers
2. **Symbolic**: starting with an operator symbol, followed by other operator symbols.

注意：

* The underscore character `_` counts as a letter.
* Alphanumeric identifiers can also end in an underscore, followed by some operator symbols.

#### 优先级

自定义的操作如优先级由其 **首字母** 决定：

```
(all letters)
|
^
&
< >
= !
:
+ -
* / %
(all other special characters)
```

越向下，优先级越高。

### `Rational` 最终版

```
class Rational(x: Int, y: Int) {
  require(y != 0, "den can't be 0!")

  private def gcd(a: Int, b: Int): Int = if (b == 0) a else gcd(b, a % b)
  private val g = gcd(x, y)

  def num: Int = x / g
  def den: Int = y / g

  def this(x: Int) = this(x, 1)

  def + (that: Rational) =
    new Rational(
      num * that.den + den * that.num,
      den * that.den)

  def - (that: Rational): Rational = this + -that

  def < (that: Rational): Boolean = num * that.den < den * that.num

  def max(that: Rational): Rational = if (this < that) that else this

  def unary_- = new Rational(-num, den)

  override def toString: String = num + "/" + den
}
```

`add` `sub` `less` 使用 `+` `-` `<` 替代，更加自然。

> 注意 unary_ 用来定义单操作数函数。
