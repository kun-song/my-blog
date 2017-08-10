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

因此匿名函数是普通函数的 **语法糖**，任意匿名函数都可以用普通函数替代。

### 匿名函数作为参数

现有函数：

```
def sum(f: Int => Int, a: Int, b: Int): Int =
  if (a > b) 0
  else f(a) + sum(f, a + 1, b)
```
第一个参数为函数，既可以传入普通函数，也可以传入匿名函数：

```
// 1. 传入普通函数
def fact(x: Int): Int = if (x == 0) 1 else fact(x - 1)
def sumFactorials(a: Int, b: Int) = sum(fact, a, b)
// 2. 传入匿名函数
def sumFactorials(a: Int, b: Int) = sum((n: Int) => if (n == 0) 1 else n * fact(n - 1), a, b)
```

## 柯里化（语法糖）

```
def sum(f: Int => Int): (Int, Int) => Int = {
  def sumF(a: Int, b: Int): Int =
  if (a > b) 0
  else f(a) + sumF(a + 1, b)
  sumF
}
```

`sum` 函数接受函数 `f`，且返回一个函数，所以可以这样使用：

```
def sumInts = sum(x => x)
def sumCubes = sum(x => x * x * x)
def sumFactorials = sum(fact)
sumCubes(1, 10) + sumFactorials(10, 20)
```
* `sumInts` ... 是 `sum` 函数返回的函数；

> The definition of functions that return functions is so useful in functional programming that there is a special syntax for it in Scala.

```
def sum(f: Int => Int)(a: Int, b: Int): Int =
  if (a > b) 0 else f(a) + sum(f)(a + 1, b)
```
