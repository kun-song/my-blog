# 第一周：基础

## 1. 编程范式（Programming Paradigms）

> Paradigms: In science, a paradigm describes `distinct concepts` or `thought patterns` in some scientific discipline.

目前有 3 种主要的编程范式：

1. 命令式
2. 函数式
3. 逻辑式

有人认为面向对象也是一种编程范式，但此处 Martin 并不赞同，他认为面向对象与上面三种范式是 **正交** 的，即它可以与上面任意一种范围 **相结合**。

### 命令式

命令式主要包括：

1. modifying mutable variables
2. using assignments
3. control structures
  * if-then-else
  * loops
  * break
  * continue
  * return

可以将命令式的程序理解为发送给 *冯诺依曼计算机** 的 **一系列指令集合**。指令式范式中的许多概念直接对应于计算机底层部件、指令：

1. mutable variables         <=>   memory cells
2. variables dereferences    <=>   load 指令（可以加载指令、数据）
3. variables assignments     <=>   store 指令
4. control structures        <=>   jump 指令

#### 缺陷

##### 扩展性

指令式只能 `word by word` 编程，无法在更高抽象层次上编程，表达能力太底层，且受限于 **冯诺依曼体系**，限制了可扩展性（见 John Backus（Pascal 作者） 的图灵奖论文 Can Programming Be Liberated from the von. Neumann Style?）。

### 函数式

* 严格定义：函数式编程为不使用 mutable variables, assignments, loops 以及其他指令式 control structures 的编程范式；
  + Pure Lisp, XSLT, XPath, XQuery, FP
  + Haskell (without I/O Monad or UnsafePerformIO)
* 宽松定义：更加关注函数的编程范式；
  + Lisp, Scheme, Racket, Clojure
  + SML, Ocaml, F#
  + Haskell (full language)
  + Scala
  + Smalltalk, Ruby (!)

### 推荐教材

1. Structure and Interpretation of Computer Programs（很多课程内容、习题出自该书）
2. Programming in Scala
3. Scala for the Impatient

## 2. Scala Evaluation

一般语言会提供：

1. primitive expressions
  * 表示语言中最基本的元素
2. ways to `combine` expressions
3. ways to `abstract` expressions
  * 抽象是指给 expressions 一个名字，使用该名字可以获取该 expressions

函数式编程很像在使用 **计算器**，大多数函数式语言会提供 REPL，在 REPL 中可以快速实验，有两种方式可以进入 Scala REPL，两者效果相同：

1. `scala`
2. `sbt console`

### Scala 求值规则

对于 non-primitive expression，Scala 规则如下：

1. Take the leftmost operator
2. Evaluate its operands (left before right)
3. Apply the operator to the operands

对于 name 求值，规则如下：

1. A name is evaluated by `replacing` it with the `right hand side` of its definition

求值过程在遇到结果为 `value` 时停止。

对于函数求值，规则如下：

1. Evaluate `all` function arguments, from left to right
2. `Replace` the function application by the function’s right-hand side, and, at the same time `Replace` the formal parameters of the function by the actual arguments.

### substitution model

Scala 通过 `substitution model` 进行表达式求值：

* substitution model 将求值抽象为 `reduce an expression to a value`
* 该模型可用于所有无副作用的表达式。
* substitution model 是函数式编程的基础。

### evaluation strategy

* call-by-value
  + 即在对函数求职之前，首先计算参数的 **值**，然后用 **函数体** 替换函数名
* call-by-name
  + 先替换函数体，最后替换参数名对应的 expression

当函数为纯函数，且两种求值都能正常结束时，两种方式将归约到同一个值。

两种求值策略各有优势：

* call-by-value 保证 expression 只计算一次
* call-by-name 保证不计算 **未使用** 的 expression

### 例子

现有如下函数：

```
def test(x: Int, y: Int): Int = x * x;
```

对于 `test(2, 3)` 两种求值策略步骤相同：

```
test(2, 3)
2 * 2
4
```

对于 `test(3+4, 8)`，call-by-value 步骤更少，因为 call-by-name 有重复计算：

```
test(3+4, 8)
test(7, 8)
7 * 7
49

test(3+4, 8)
(3+4) * (3+4)
7 * (3+4)
7 * 7
49
```

对于 `test(7, 2*4)`，call-by-name 步骤更少，因为它不会计算未使用的表达式（即 2*4）：
```
test(7, 2*4)
test(7, 8)
7 * 7
49

test(7, 2*4)
7 * 7
49
```

对于 `test(3+4, 2*4)` 两者步骤数量相同，但每步的内容不一定相同：

```
test(3+4, 2*4)
test(7, 2*4)
test(7, 8)
7 * 7
49

test(3+4, 2*4)
(3+4) * (3+4)
7 * (3+4)
7 * 7
49
```

### 求值策略、终止

> call-by-name and call-by-value evaluation strategies reduce an expression to the `same value`, as long as both evaluations `terminate`.

如果求值过程可能无法终止，则：

* If CBV evaluation of an expression e terminates, then CBN evaluation of e terminates, too.
* The other direction is not true

Scala 默认采用 call-by-value，但也可以使用 call-by-name（使用 `=>` 操作符）：

```
def constOne(x: Int, y: => Int) = 1
```
* x: call-by-value
* y: call-by-name

`constOne(1+2, loop)` 将正确终止（因为第二个参数为 call-by-name，而函数体中没有使用），而 `constOne(loop, 1+2)` 将无法终止。

### 3. 条件表达式、值定义

Scala 的 `if-else` 是表达式，并非语句（与 Java 不同）。

Scala 的值定义分为两种：by-name & by-value。

#### definition by name

使用 `def` 关键字，右侧表达式在 **使用时** 进行计算（每次都要重新计算）。

#### definition by value

使用 `val` 关键字，右侧表达式在 **定义时** 立即计算，计算完成后，值名字将绑定到右侧的计算结果值。

#### 例子

现有函数定义如下：

```
def loop: Boolean = loop
def x = loop
val y = loop
```

* `x` 只是定义了 `loop` 的另一个名字，并不立即计算；
* `y` 在定义时计算 `loop`，发生死循环；

### 使用牛顿下山法计算平方根

```
def abs(x: Double) = if (x < 0) -x else x
/**
  * 1. 非常小的值，平方根错误
  * 2. 非常大的值，无法终止
  */
def isGoodEnough(guess: Double, x: Double) = abs(guess * guess - x) < 0.001
def isGoodEnoughBetter(guess: Double, x: Double) = abs(guess * guess - x) / x < 0.001

def improve(guess: Double, x: Double) = (guess + x / guess) / 2

def sqrtIter(guess: Double, x: Double): Double =
  if (isGoodEnoughBetter(guess, x)) guess
  else sqrtIter(improve(guess, x), x)

def sqrt(x: Double): Double = sqrtIter(1.0 , x)
```

### Block

Block is also a **expression**.

```
val x = 0
def f(y: Int) = y +1
val result = {
  val x = f(3);
  x * x
} + x
```

* block 最后一个表达式的值，即为整个 block 的值，在本例中，最后一个表达式为 `x * x`；
* block is expression，可以出现在任何表达式可出现的位置；

上例中的 `sqrtIter` `isGoodEnough` `improve` 等方法不应该对用户开放，可以使用 block 进行隐藏：

```
def sqrt(x: Double): Double = {
  def abs(x: Double) = if (x < 0) -x else x

  /**
    * 1. 非常小的值，平方根错误
    * 2. 非常大的值，无法终止
    */
  def isGoodEnough(guess: Double, x: Double) = abs(guess * guess - x) < 0.001
  def isGoodEnoughBetter(guess: Double, x: Double) = abs(guess * guess - x) / x < 0.001

  def improve(guess: Double, x: Double) = (guess + x / guess) / 2

  def sqrtIter(guess: Double, x: Double): Double =
    if (isGoodEnoughBetter(guess, x)) guess
    else sqrtIter(improve(guess, x), x)

  sqrtIter(1.0, x)
}
```

### Lexical Scope（闭包）

利用词法作用域改进 `sqrt`：

```
def sqrt(x: Double): Double = {
  def abs(x: Double) = if (x < 0) -x else x

  /**
    * 1. 非常小的值，平方根错误
    * 2. 非常大的值，无法终止
    */
  def isGoodEnough(guess: Double) = abs(guess * guess - x) < 0.001

  def isGoodEnoughBetter(guess: Double) = abs(guess * guess - x) / x < 0.001

  def improve(guess: Double) = (guess + x / guess) / 2

  def sqrtIter(guess: Double): Double =
    if (isGoodEnoughBetter(guess)) guess
    else sqrtIter(improve(guess))

  sqrtIter(1.0)
}
```

### 多行表达式

多行表达式用 `+` 分割，例如：

```
(A
+ B)

C +
D
```

注意如下写法将会被视为两个表达式：

```
A
+ B
```

## 尾递归

One evaluates a function application `f(e1, ..., en)`:

* by evaluating the expressions `e1, . . . , en` resulting in the values `v1, ..., vn`, then
* by replacing the application with the body of the function f, in which
* the actual parameters `v1, ..., vn` **replace** the formal parameters of f.

现有函数：

```
def gcd(a: Int, b: Int): Int = if (b == 0) a else gcd(b, a % b)
def factorial(n: Int): Int = if (n == 0) 1 else n * factorial(n - 1)
```

对于 `gcd(14, 21)`，计算过程如下：

```
gcd(14, 21)
→ if (21 == 0) 14 else gcd(21, 14 % 21)
→ if (false) 14 else gcd(21, 14 % 21)
→ gcd(21, 14 % 21)
→ gcd(21, 14)
→ if (14 == 0) 21 else gcd(14, 21 % 14)
→→ gcd(14, 7)
→→ gcd(7, 0)
→ if (0 == 0) 7 else gcd(0, 7 % 0)
→ 7
```

对于 `factorial(4)`，计算过程如下：

```
factorial(4)
→ if (4 == 0) 1 else 4 * factorial(4 - 1)
→→ 4 * factorial(3)
→→ 4 * (3 * factorial(2))
→→ 4 * (3 * (2 * factorial(1)))
→→ 4 * (3 * (2 * (1 * factorial(0)))
→→ 4 * (3 * (2 * (1 * 1)))
→→ 120
```

观察 `factorial(4)` 和 `gcd(14, 21)` 的计算过程，可以看到 `factorial` 每次递归调用，都需要保存 **上次递归** 中的值，最后表达式越来越长：`4 * (3 * (2 * (1 * 1)))`。

**尾递归**：:

* If a function calls itself as its `last action`, the function’s `stack frame` can be reused. This is called tailrecursion.
* 尾递归可以视为 **另一种形式** 的迭代，效率与循环相同；
* 尾递归的 **栈帧** 可以被多次递归的函数复用，因此只需 **常量空间**；

**尾调用**：

* In general, if the last action of a function consists of calling a function (which may be the same), one `stack frame` would be `sufficient` for both functions. Such calls are called `tail-calls`.
* 尾调用其实也只需 **一个栈帧**；
* 尾递归是尾调用的特例；

### Scala 中的尾递归

* 在 Scala 中，**只有** 尾递归才能被优化（尾调用无法优化）；
* 使用 `@tailrec` 修饰的函数，**必须尾递归**；
