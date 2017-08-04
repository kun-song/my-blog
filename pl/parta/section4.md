# 第四部分：类型推断、模块系统、相等性

## 类型推断

### 什么是类型推断？

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

ML 类型检查，给定 ML 变量、表达式，ML 推断出他们的类型，使得 **类型检查** 能够通过。

### ML 中的类型推断

ML 类型推断有以下步骤：

1. `Determine types of bindings in order`
  * `mutual recursion` 除外
  * 不能使用 **后面** 的绑定（无法通过类型检查）
2. 对于 `val` `fun` 绑定：
  * 分析定义，获取约束，比如看到 `x > 0`，则推断出 `x` 为 `int` 类型，看到 `x / 2.0` 可推断出 `x` 为 `real` 类型；
  * 若 **约束过多**，无法保持，则发生类型错误；
3. 对于 **无约束** 的类型，使用类型变量（`'a`）表示
  * 未使用的函数参数
4. 强制 `value restriction`

### 多态类型推断

```
fun length xs =
  case xs of
    [] => 0
  | x :: xs' => 1 + length xs';  
```

1. 每个函数类型都是 `T1 -> T2`
2. `x::xs'` 模式可以推断 `xs'` 是 `list`，假设 `x` 为 `T3` 则 `xs'` 为 `T3 list`；
3. `length xs'` 说明 `T1 = T3 list`
4. `[]` 分支结果为 0，说明函数返回值为 `int`，即 `T2 = int`
5. 到目前为止，函数类型为 `T3 list -> int`

最后无任何约束进一步确认 `T3` 的类型，实际上它可以使 **任意类型**，所以 `length` 的类型为 `'a list -> int`

```
fun compose (f, g) = fn x => f (g x);
```

1. 函数类型为：`T1 * T2 -> T3`
2. 假设 `x` 类型为 `T4`，则 `g` 类型为 `T4 -> T5`，`f` 类型为 `T5 -> T6`
3. 因为 `f` 的返回值为 `compose` 的返回值，所以 `T3 = T6`
4. 所以 `compose` 类型为：`(T5 -> T6) * (T4 -> T5) -> T4 -> T6`

使用 **类型参数** 替换后，`compose` 类型为 `('c -> 'b) * ('a -> 'c) -> 'a -> 'b`。
