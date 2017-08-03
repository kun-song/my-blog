# 第四部分：类型推断、模块系统、相等性

## 类型推断

编程语言分为：

1. 静态类型：在编译时进行 **类型检查**
  * ML Haskell Scala Java
2. 动态类型：在运行时进行 **类型检查**
  * Racket JavaScript Ruby

ML 是静态类型语言，但是 ML 一般是 `implicitly typed`，即 **无需显式写出类型**，由类型推断处理，例如：

```
fun f x =
  if x > 3
  then x
  else x * 2
```
* ML 能自动推断出 `f` 的类型为 `int -> int`

下面的例子无法通过类型检查：

```
fun f x =
  if x > 3
  then true
  else x * 2
```
