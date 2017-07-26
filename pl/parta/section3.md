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

### 参数多态（泛型）

`n_times` 函数的类型为 `('a ->'a) * int * 'a -> a'`，其第一个参数 `f` 类型为 `'a ->'a`，该参数为多态参数，所以可以传入 `incr` `tl` 等不同类型的参数。

高阶函数经常具备多态参数，但并非强制，具备多态参数将大大增强高阶函数的威力。

## 匿名函数

高阶函数的函数参数，经常使用匿名函数，因为该函数只用于调用，而不用在其他地方。

```
fun double_n_times (n, x) = n_times (fn y = > y + y, n, x);
```

> **注意**：
* 匿名函数无法用于 **递归**，一般只作为高阶函数的参数传入；
* 若不需要递归，则 **函数绑定** 可以视为 `val` + `fn` 的语法糖；

```
// fun binding 可以视为 val + fn 的语法糖
fun double x = x + x;
val double2 = fn x => x + x;  // 太长，不推荐
```

### 不要滥用匿名函数

```
fun n_times (f, n, x) =
  if n = 0
  then x
  else f (n_times (f, n-1, x));
```

若 `f` 为截取列表的尾部，则有两种写法：

```
// 传入匿名函数
fun tail_n_times = n_times(fn y => tl y, n, x);
// 直接传入 tl 函数
fun tail_n_times = n_times(tl, n, x);
```

形如 `fn x => f x` 的匿名函数，其实是冗余的，其效果与直接调用 `f` 完全等价，不要使用这类 **过度包装** 的函数。

## Map & Filter

简单实现一个 `map` 函数：

```
fun map (f, xs) = (* 在列表 xs 的每个元素上应用函数 f，返回 f x 组成的列表 *)
  case xs of
      [] => []
    | x :: xs' => (f x) :: map(f, xs');
```
* 该函数的类型为：`('a -> 'b) * 'a list -> 'b list`

```
fun filter (f, xs) = (* 遍历 xs，取 f x 为 true 的元素，组成新列表 *)
  case xs of
      [] => []
    | x :: xs' => if f x
		              then x :: filter(f, xs')
		              else filter(f, xs');

```
* 该函数类型为：`('a -> bool) * 'a list -> 'b list`

## 函数作为返回值

```
(* double_or_triple 接受函数 f 作为参数，并且返回一个函数 *)
fun double_or_triple f =
  if f 7
  then fn x => x * 2
  else fn x => x * 3;

(* 调用 double_or_triple 获取返回函数 *)
val double = double_or_triple (fn x => x = 7);
val triple = double_or_triple (fn x => x <> 7);

val test1 = double 2 = 4;
val test2 = triple 2 = 6;
```

> 注意：double_or_triple 类型为 (int -> bool) -> int -> int，由于 `->` 是右结合的，所以等价于 (int -> bool) -> (int -> int)。

## 高阶函数用于自定义 `datatype`

```
datatype exp = Constant of int
	     | Negate of exp
	     | Add of exp * exp
	     | Multiply of exp * exp;

fun true_of_all (f, ex) =
  case ex of
      Constant i => f i
    | Negate i => true_of_all (f, i)
    | Add (x, y) => true_of_all (f, x) andalso true_of_all (f, y)
    | Multiply (x, y) => true_of_all (f, x) andalso true_of_all (f, y);


fun all_even e = true_of_all (fn x => x mod 2 = 0, e);

val test1 = all_even (Add (Constant ~1, Negate (Constant 2))) = false;
val test2 = all_even (Add (Multiply(Constant 4, Constant 2), Negate (Constant 2))) = true;

fun all_positive e = true_of_all (fn x => x > 0, e);

val test1 = all_positive (Add (Constant ~1, Negate (Constant 2))) = false;
val test2 = all_positive (Add (Multiply(Constant 4, Constant 2), Negate (Constant 2))) = true;
```

* 高阶函数适用于 **递归数据结构**

## 词法作用域 `lexical scope`

函数可以使用 `scope` 中所有绑定，但这个 `scope` 具体是指什么呢？函数有两个重要阶段：

1. 定义阶段
2. 调用阶段

### 闭包

> Functions are values.

但是 `Function Value` 到底是什么样的值呢？

ML 称 **function value** 为 `function closure` 或者 `closure`，包含两部分：

1. `code` 即函数代码
2. `environment` 即函数 **定义时** 的环境

这类似 ML 中的 `pair`，但无法分别访问这两部分，使用该 `pair` 的唯一方式为：函数调用，即：

* 在 `environment` + 函数参数扩展 中
* 执行 `code`
