# 第三部分：第一等函数、闭包、高阶函数

## 函数式编程

特点：

1. 使用不可变数据
2. 函数作为值
3. 多使用递归、递归数据结构
4. 编程风格类似数学中的函数定义
5. 惰性求值（Haskell）

### First-Class Functions & Function Closure

`First-class function` 即： **函数是第一等公民**。

* `Functions are values too`
* 函数可以出现在任何 **值** 可以出现的地方：
  1. 函数参数、函数返回结果（最常用）
  2. `tuple` 的元素
  3. 变量绑定
  4. 作为 `datatype constructor` / `exception` 携带的数据

```
fun double x = x + x;
fun incr x = x + 1;
(*
   1. function is value.
   2. double incr 都是函数，与普通值并没有区别，所以可以出现在任何 value 可以出现的地方；
   3. a_tuple 的类型为：(int -> int) * (int -> int) * int
*)
val a_tupe = (double, incr, double (incr 7));
(*
   1. 提取 a_tuple 中保存的 function，并调用该函数
*)
val eighteen = #1 a_tupe 9;
```

`Function Closure` 是指函数可以使用 **函数体外定义** 的绑定，与 `High Order Function` 结合后，威力无穷。

> First-class function 与 Function Closure 定义完全不同，但函数式语言，比如 ML，经常同时支持两者，所以两个经常混用。

### 函数作为参数

```
fun f (g, ...) = ... g ()
```
* 函数 `g` 作为 `f` 的参数传入，在 `f` 函数体内，可以调用函数 `g (...)`；
* 这对 **抽象** 意义重大，只需要将 **不同部分** 抽取为函数，在调用时传入即可，而大部分操作都可以复用；
