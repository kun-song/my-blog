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

### Value Restriction

到目前为止的类型推断，都是 **不完善** 的，即通过类型检查之后，并不能保证类型正确，`polymorphism` + `mutable reference` 一起就会产生该问题，例如：

```
val r = ref NONE;
val _ = r := SOME "Hi";
val i = 1 + ValOf(!r);
```
* `r` 类型应该为 `'a option ref`（实际上是 `?.X1 option ref`）；
* 第二行对 `r` 赋值为 `string option ref`；
* 第三行实际为 `int` + `string`，类型错误；

编译器对第一行有如下提示：

```
value_restriction.sml:3.5-3.17 Warning: type vars not generalized because of
   value restriction are instantiated to dummy types (X1,X2,...)
val r = ref NONE : ?.X1 option ref
```

为了保持完备性，需要 **增强** 类型系统，因为 ML 类型系统 **无法区分** `mutable reference`，所以无法针对 `reference` 做特殊处理。

ML 通过 value restriction 解决该问题，即：

> `Value Restriction`: a variable-binding can have a **polymorphic type** only if the expression is a **value** or `variable`.
* Ref NONE 是函数调用，所以不能是泛型的。

如果没有使用 `mutable reference`，按理说不需要 `value restriction`，但 ML 无法区分，所以会造成不便：

```
val f = List.map (fn x => (x, 1));
```

按理说，上面的 `f` 类型应该是 `'a list -> ('a * int) list`，但由于 `value restriction`，右边的表达式不是 `value` 也不是 `variable`，所以最后 `f` 函数使用 `dummy types` 定义，结果就是编译时警告，同时后面 **无法使用** 该函数。

#### 使用 function-binding 替代 variable-binding

但这并不意味着无法实现该函数，因为 `value restriction` 仅在 `variable-binding` 时起作用，使用 `function-binding` 即可：

```
fun f xs = List.map (fn x => (x, 1)) xs;
```
* `f` 类型为 `'a list -> (a' * int) list`，可 **保持多态**；

#### 指定类型

```
val f: int list -> (int * int) list = List.map (fn x => (x, 1));
```
* 将 `f` 的类型指定为 `int list -> (int * int) list`，可以通过类型检查，但 **丧失多态**；

## Mutual Recursion

在 ML 中，只能使用 **前面** 定义的绑定，所以无法实现 `f` 调用 `g`，同时 `g` 调用 `f`（两者皆为函数）。ML 有两者方式可以显示 `mutual recursion`：

* `and` 关键字
* 高阶函数（仅作为示例，效率较低）

### `and` 关键字

由 `and` 关键字定义的 `mutual recursion bundle` 将会合在一起进行 **类型检查**，所以它们可以 **互相递归调用**。

#### `and` 用于定义 `mutual recursive function`

```
fun f1 p1 = e1
and f2 p2 = e2
and f3 p3 = e3
```
* 将后面所有 `fun` 关键字替换为 `and`

#### `and` 用于定义 `mutual recursive datatype`

```
datatype t1 = ...
and t2 = ...
and t3 = ...
```
* 将后面所有 `datatype` 关键字替换为 `and`

### 惯用法：有限状态机

接受输入，每接受一个输入，就计算出对应的状态机的状态：

```
fun match xs =
  let
      fun s_need_one s =
	case xs of
	    [] => true
	  | 1 :: xs' => s_need_two xs' (* mutual recursion *)
	  | _ => false
      and s_need_two xs =
	  case xs of
	      [] => false
	    | 2 :: xs' => s_need_one xs' (* mutual recursion *)
	    | _ => false
  in
      s_need_one xs
  end;
```

## 模块系统

### 模块定义

对于大型程序，模块化是必然要求，ML 使用 `structure` 绑定定义模块：

```
structure ModuleName = struct bindings end;
```

* `structure ... = struct ... end`
* `ModuleName` 为模块名字
* `bindings` 可以是任何绑定，包括 `val` `fun` `datatype` `exception` ...

在模块内部，以前的定义均有效，在模块外部，要访问 `ModuleName` 模块中定义的绑定，需要用 `ModuleName.bindingName`（例如 `List.map`）。

```
structure MyMathLib =
struct
fun fact x = if x = 0
	      then 1
	      else x * fact(x-1)
val half_pi = Math.pi / 2.0
fun doubler x = x + x
end;

val pi = MyMathLib.half_pi + MyMathLib.half_pi;
val v28 = MyMathLib.doubler 14;
```
* `MyMathLib` 模块定义了 3 个绑定，在模块之外，通过 `MyMathLib.fact` 等进行访问；
* `MyMathLib` 并非表达式、并非值，在 REPL 直接输入 `MyMathLib` 报错：`unbound variable or constructor: MyMathLib`；

#### `open` 关键字

使用 `open ModuleName` 将暴露 `ModuleName` 中定义的所有绑定：

```
open MyMathLib;
val v28' = doubler 14;
```
* 一般仅将 `open` 用于测试。
* 还可以用 `val doubler = MyMathLib.doubler` 简化书写。

### `Signature` & Hide Things

#### `signature` 使用

`signature` 是模块的 **类型**，定义了：

1. 模块必须包含的绑定
2. 绑定的 **类型**

```
(* signature ： 模块的类型 *)
signature MATHLIB =
sig
    val fact: int -> int
    val half_pi: real
    val doubler: int -> int
end;

(* :> 符号表明，MathLib 模块必须具备 MATHLIB 签名声明所有绑定，且类型必须正确 *)
structure MathLib :> MATHLIB =
struct
fun fact x = if x = 0 then 1 else x * fact(x-1)
val half_pi = Math.pi / 2.0
fun doubler x = x + x
end;

(* test cases *)
val v28 = MathLib.doubler 14 = 28;
```

* `signature` 是模块的类型，其中用 `val` 声明一系列 **变量**（这些变量通过什么方式绑定无所谓，可以用 `variable-binding` or `function-binding` ...）
* `:>` 为模块声明签名

#### Hiding Things

##### 函数隐藏

```
fun doubler x = x + x;
fun doubler x = x * 2;
val y = 2;
fun doubler x = x * y;
```

这 3 个函数，对于使用者来说无法区分，可以互相替换，即函数隐藏了其内部实现。

##### 局部函数

局部函数对于 `top-level` 是不可见的，也算是一种隐藏。

#### `top-level` 级别的隐藏

`top-level` 级别的 *私有* 函数非常有用，比如两个函数需要使用同一个辅助函数，则将该辅助函数定义为 `top-level private` 会方便很多，ML 通过在 `signature` 中 **遗漏** 部分绑定来实现 `private` 效果：

```
(* signature ： 模块的类型 *)
signature MATHLIB =
sig
    val fact: int -> int
end;

(* :> 符号表明，MathLib 模块必须具备 MATHLIB 签名声明所有绑定，且类型必须正确 *)
structure MathLib :> MATHLIB =
struct
fun fact x = if x = 0 then 1 else x * fact(x-1)
val half_pi = Math.pi / 2.0
fun doubler x = x + x
end;
```
* `signature` 中只有一个变量 `fact`，则模块中的 `half_pi` `doubler` 都是 `top-level` 的 `private` 变量，在模块之外，无法访问；

### 例子

```
structure Rational1 =
struct

datatype rational = Whole of int | Frac of int * int;
exception BadFrac;

fun gcd x y =
  if x = y
  then x
  else
      if x < y
      then gcd x (y-x)
      else gcd y x;

fun reduce r =
  case r of
      Whole _ => r
    | Frac (x, y) => if x = 0
		    then Whole 0
		    else let val d = gcd (abs x) y
			 in
			     if d = y
			     then Whole(x div d)
			     else Frac(x div d, y div d)
			 end;

fun make_frac x y = if y = 0
		    then raise BadFrac
		    else
			if y < 0
			then reduce(Frac(~x, ~y))
			else reduce(Frac(x, y));

fun add (r1, r2) =
  case (r1, r2) of
      (Whole i, Whole j) => Whole(i+j)
    | (Whole i, Frac(j, k)) => Frac(i*k + j, k)
    | (Frac(j, k), Whole i) => Frac(i*k + j, k)
    | (Frac(a, b), Frac(c,d)) => reduce(Frac(a*d + c*b, d*d));

fun toString r =
  case r of
      Whole i => Int.toString i
    | Frac(a, b) => (Int.toString a) ^ "/" ^ (Int.toString b);
end;
```

#### 最直观的 `signature`

有以上模块实现，其中 `gcd` `reduce` 为内部辅助函数，用户无需感知，则其类型可以写成：

```
(*
   client 可能会错误的直接使用 rational 的构造函数，但若简单的移除构造函数，类型检查错误
*)
signature RATIONAL_A =
sig
    datatype rational = Whole of int | Frac of int * int
    exception BadFrac
    val make_frac: int * int -> rational
    val add: rational * rational -> rational
    val toString: rational -> string
end;
```

该签名，不包含 `gcd` `reduce` 变量，所以用户确实无法使用它们，但其有一个 **致命错误**：用户可以直接使用 `rational` 的构造函数，从而构造出不符合规范的数字，我们本意是用户只能通过 `make_frac` 来产生 `rational` 类型的值。

直观的解决方法为，在 `signature` 中去掉 `rational` 的定义，但这无法通过类型检查，因为 `add` 等函数使用了名为 `rational` 的类型，但该类型没有定义。

#### 含有 `abstract type` 的 `signature`

ML 针对这种场景，提供了 `abstract type`（仅用于 `signature` 定义中）：

```
type foo;
```

> * 告知用户存在类型 `foo`，但是用户无法得知其定义（在本例中，进而只能通过 `make_frac` 定义 `rational` 类型的值）；
> * 以前使用 `type` 关键字定义类型 **同义词**：`type foo = bar`；

```
(*
   rational 被定义为 abstract type，则 add 等可以继续使用 rational，同时 client 无法使用其构造函数
*)
signature RATIONAL_B =
sig
    type rational (* abstract type *)
    exception BadFrac
    val make_frac: int * int -> rational
    val add: rational * rational -> rational
    val toString: rational -> string
end;
```

使用 `abstract type` 后，`add` 等函数可以使用 `rational` 类型，同时用户也无法使用 `rational` 的构造函数，perfect ~~

#### 暴露指定构造函数的 `signature`

`signature` 还可以暴露指定的构造函数，若用户需要直接使用 `Whole` 创建 `rational` 值，则 `signature` 如下：

```
signature RATIONAL_B =
sig
    type rational (* abstract type *)
    exception BadFrac
    val Whole: int -> rational (* 构造函数 -> 也是函数 *)
    val make_frac: int * int -> rational
    val add: rational * rational -> rational
    val toString: rational -> string
end;
```

> 用户无法感知 `Whole` 是构造函数，用户认为其为普通函数。

### `structure` 匹配 `signature` 的规则

若有 `structure Foo :> BAR ...`，则若满足以下条件，该模块可以通过类型检查：

1. `BAR` 中所有 `non-abstract type`，在 `Foo` 中都有，且与 `BAR` 中定义方式 **保持一致**；
2. `BAR` 中所有 `abstract type`，在 `Foo` 中都有，定义方式可以是 `datatype` 或者 `type` 同义词；
3. `BAR` 中所有 `val-binding`，在 `Foo` 中都有，但：
  * `Foo` 中的版本类型可以更加宽松（**泛型**）；
  * 可以通过 `variable-binding` `function-binding` ...
4. `BAR` 中所有 `exception`，在 `Foo` 中都有；

> 另外：`Foo` 中可以含有 **更多** 绑定；

## 函数相等

给定两个函数，它们相等的条件为：在任意使用场景下，若：

1. 计算结果相同；
2. 终止条件、非终止条件相同；
3. 修改相同内存；
4. 输入、输出相同；
5. 抛出异常相同；

例子1：

```
fun f1 x = x + x;

val y = 2;
fun f2 x = x * y;
```

* 函数 `f1` `f2` 相等；

例子2：

```
fun g1 (f, x) = (f x) + (f x);

val y = 2;
fun g2 (f, x) y * (f x);
```

* 若 `f` 函数有副作用，比如IO，则两者行为不同，所以不相等；

例子3：

```
fun f x =
  let
    val y = g x
    val z = h x
  in
    (y, z)
  end;

fun f x =
  let
    val z = h x
    val y = g x
  in
    (y, z)
  end;
```

* 同样，若 h g 有副作用，比如 IO、引用读写、异常抛出等，两者行为不同，所以不相等；

> 可以看到，`side-effect`（包括：IO、`mutable reference`、异常等）很容易导致函数不相等，ML 推荐 `side-effect free` 风格，从而获得更广泛的相等性。

### `Standard Equivalences`

有几种情况，可以确保函数相等。

#### 语法糖

按照定义，语法糖可以互相替换，否则就不是语法糖：

```
fun f x =
  if x
  then g x
  else false;

fun f x = x andalso g x;
```
* 两者等价，因为 `x andalso g x` 是 `if x then g x else false` 的语法糖；
* 但 `x andalso g x` 与 `g x andalso x` 不同，因为 `g x` 可能有副作用，或者可能无法停止；

#### 修改变量名

```
fun x = x + 14;

fun y = y + 14;
```

* 只要 `y` 没有被使用过，可以将 `x` 替换为 `y` 并保证相等（几乎所有语言都可以这样做）；

#### 本地辅助函数

是否使用本地辅助函数不会影响相等性：

#### 过度包裹的函数

```
fun f x = g x;

val f = g
```

#### `let` 表达式与匿名函数

```
let val x = e1 in e2 end;

(fn x => e2) e1;
```

* 两者行为完全一致：都是先计算 `e1`，然后将其模式匹配到 `x`，然后计算 `e2`；
* 在 Racket 中，两者完全一致，但在 ML 中有一点区别：
  1. `let` 中的 `x` 可以是多态的；
  2. `fn` 中的不可以；
