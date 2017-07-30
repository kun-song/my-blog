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

### `lexical scope` 优势

编程语言中有两种作用域：

1. `lexical scope`：使用 **函数定义** 时的 `environment`
2. `dynamic scope`：使用 **函数调用** 时的 `environment`

十几年前，大家认为这两种作用域都是合理的，但到今天，编程语言普遍采用 `lexical scope`，很少有采用 `dynamic scope` 的语言。

### 闭包 & 重复计算

* **函数调用** 时会计算函数体
* **每次** 函数调用都会计算函数体
* 变量绑定，只有在 **该绑定被计算时** 才会计算其绑定的 **表达式**，并非每个使用该绑定就会计算其表达式

有了以上三点，可以减少重复计算。

```
fun filter (f, xs) =
  case xs of
      [] => []
    | x :: xs' => if f x
		  then x :: filter(f, xs')
		  else filter(f, xs');

(* 对于 xs 中的每个元素，传入的闭包都要计算一次，则 String.size s 也需要重复计算很多次 *)
fun allShorterThan1 (xs, s) = filter(fn x => String.size x < (print "!"; String.size s), xs);

(* 如下所示，l 只会计算一次 *)
fun allShorterThan2 (xs, s) =
  let
      val l = (print "!"; String.size s)
  in
      filter (fn x => String.size x < l, xs)
  end;
```

## `Fold` 函数

`fold` 函数可能是除了 `map` `filter` 以外，最有用的高阶函数之一。一个简单实现如下：

```
fun fold (f, acc, xs) =
  case xs of
    [] => acc
  | x :: xs' => fold(f, f(acc, x), xs');
```
* 注意 `f(acc, x)` 很清晰说明了 `fold` 函数的行为：不断迭代 `xs` 的每个元素，最后产生一个新的 `acc` 返回；

### 分离 `recursive traversal` 与 `data processing`

到目前为止，`map` `filter` `fold` 三个函数，代码非常相似：

```
fun map (f, xs) =
  case xs of
    [] => []
  | x :: xs' => (f x) :: map(f, xs');

fun filter (f, xs) =
    case xs of
      [] => []
    | x :: xs' => if f x
                  then x :: filter(f, xs')
                  else filter(f, xs');

fun fold (f, acc, xs) =
  case xs of
    [] => []
  | x :: xs' => fold(f, f(acc, x), xs');
```

* `map` `filter` `fold` 在有的语言中，是内置特性，但在 ML 中非常容易实现；
* 三者 **模式** 非常相似，都是 **递归遍历** + **数据处理** 的模式，其中 `map` `filter` `fold` 函数实现了递归遍历，而传入的 **闭包参数** `f` 实现了对数据的处理；
* `recursive traversal` 和 `data processing` 部分都可以被 **重用**；

## 组合函数

```
(*
   f g 都是函数，通过 compose 函数组合在一起
   ('a -> 'b) * ('c -> 'a) -> 'c -> 'b
   接受 g 的参数，获得 f 的结果；g 只是中间阶段
*)
fun compose (f, g) = fn x => f(g x);

fun sqrt_of_abs1 i =  Math.sqrt (Real.fromInt (abs i));

(* o 与 compose 函数的作用相同 *)
fun sqrt_of_abs2 i = (Math.sqrt o Real.fromInt o abs) i;

val sqrt_of_abs = Math.sqrt o Real.fromInt o abs;

val test1 = sqrt_of_abs1 ~9;
val test2 = sqrt_of_abs2 ~9;
val test3 = sqrt_of_abs ~9;
```

* `o` 用于组合函数，效果与 `compose` 函数相同；

> `Math.sqrt o Real.fromInt o abs` 为 **从右向左** 结合，即先计算 `abs`，将结果传入 `Read.fromInt` 进行计算 ... 最后计算 `Math.sqrt`；

### left -> right 流水线

`o` 按照 `right -> left` 组合函数，即：

* 获取右边表达式的计算结果，作为参数，传入左边函数 ...

但程序员更加习惯 `left -> right` 的组合方式，比如 `F#` 提供的 `|>` 操作符，就是从左到右组合的，在 ML 中很容易定义该操作符：

```
(* 定义 right -> left 组合函数 *)
infix !>;
fun f !> g = g o f; (*  与 g f 等价 *)

val sqrt_of_abs4 = abs !> Real.fromInt !> Math.sqrt;
```

`!>` 按照 `left -> right` 组合函数，即：

* 获取左边计算结果，作为参数，传入右边函数 ...

## 柯里化

ML 中每个函数 **只有一个参数**，以前使用 `tuple` 模拟了 **多个参数** 的效果，现在使用柯里化实现多参函数。

```
val sorted3 = fn x => fn y => fn z => z >= y andalso y >= x;
val test = ((sorted3 3) 7) 11 = true;
```

计算过程：

1. 调用 `sorted3 3`，返回一个闭包：
  * `code`: `fn y => fn z => z >= y andalso y >= x`
  * `environment`: maps x to 3
2. 调用 1 返回的闭包，再次返回一个闭包：
  * `code`: `fn z => z >= y andalso y >= x`
  * `environment`: maps x to 3, maps y to 7
3. 调用 2 返回的闭包，得到最后结果 `true`

### 语法糖

#### currying 函数调用

函数调用可以去掉括号：

```
sorted3 3 7 11
```

#### currying 函数定义

```
val sorted3_nicer = fn x y z => ...
// or
fun sorted3_nicer x y z = y >= x ...
```

#### 对比

使用语法糖之前：

```
val sorted3 = fn x => fn y => fn z => z >= y andalso y >= x;
val test = ((sorted3 3) 7) 11 = true;
```

使用语法糖之后：

```
val sorted3 x y z => z >= y andalso y >= x;
val test = sorted3 3 7 11 = true;
```

使用语法糖的代码更加精简，但原始版本更容易理解真实的运算过程。

## 部分应用

前面的柯里化仅仅用于模拟 **多参函数**，需要使用与定义时 **相同数量** 的参数进行调用；但柯里化函数，可以用 **部分参数** 进行调用，称为部分应用（`partial application`）。

* 部分应用可以用于 **所有** 柯里化函数；

```
fun sorted3 x y z = z >= y andalso y >= x;

fun fold f acc xs =
  case xs of
      [] => acc
    | x :: xs' => fold f (f(acc, x)) xs';

fun range i j = if i > j then [] else i :: range (i+1) j;

fun exists predicate xs =
  case xs of
      [] => false
    | x :: xs' => predicate x orelse exists predicate xs';

(* 提供 fewer 参数调用：部分应用 *)

val is_nonegative = sorted3 0 0; (* 不要写成 fun is_nonegative x = sorted 0 0 x *)
val sum = fold (fn (acc, x) => acc + x) 0; (* 不要写成 fun sum xs = fold (fn (acc, x) => acc + x) 0 xs *)
val countup = range 1; (* 不要写成 fun countup x = range 1 x *)
val has7 = exists (fn x => x = 7);
```

> `val sum = fold (fn (acc, x) => acc + x) 0` 使用部分应用，非常简洁，不要写成函数调用形式：`fun sum xs = fold (fn (acc, x) => acc + x) 0`；因为形如 `fun f x = g x` 的函数定义都可以直接写作 `val f = g`

### Value Restriction

柯里化 + 部分应用有时会产生 `type vars not generalized` 警告：

```
// 产生 type vars not generalized 警告，无法调用 pairWithOne 函数
val pairWithOne = List.map (fn x => (x, 1));
```

解决办法有两种：

```
// 1. 使用 fun 绑定，而非 val 绑定
fun pairWithOne xs = List.map (fn x => (x, 1)) xs;

// 2. 使用 val 绑定，但明确指定类型
val pairWithOne: string list -> (string * int) list = List.map (fn x => (x, 1));
```

## 回调函数


##
