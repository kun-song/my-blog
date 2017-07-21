# 第二部分：类型定义

目前已经有：

1. `base type`：比如 `int` `bool` `real`
2. 可以构建 `compound type`：比如 `tuple` `list` `option`，可以包含 `base type` 类型的数据

要构建更多的 `compound type` `t`，通常有三种方法：

1. `each of`
  * 类型 `t` 包含值 `v1` **and** `v2` **and** `v3` ...，其类型分别为 `t1` and `t2` and ...
2. `one of`
  * 类型 `t` 包含值 `v1` **or** `v2` **or** `v3` ...，其类型分别为 `t1` or `t2` or ...
3. `self reference`
  * `t` 类型的值可以包含其他 `t` 类型的值（递归）。

举例：

* `tuple` 是 `each of` 类型，比如 `int * bool` 包含 `int` 值 **and** `bool` 值。
* `option` 是 `one of` 类型，比如 `int option` 包含 `int` 值 **or** 不包含任何值。
* `list` 涉及上述三种：
  1. `int list` 包含 `int` 值 **and** 另一个 `int list` **or** nothing。

## 使用 `record` 构建 `each of` 类型

使用 `tuple` 可以构建 `each of` 类型，而 `record` 与此很类似。

```
val x = {bar = (1 + 3, true andalse true), foo = 1 + 5, baz = (false, true)}
```

* `x` 类型为 `{bar: int * bool, foo: int, baz: bool * bool}`，注意类型在定义 `x` 时 **无需显式指明**，`type-checker` 会自动推导 `x` 的类型，所以直接写值即可。
* 字段名可以有 **任意顺序**，最后的 **类型相同**

### `record` 用法

`record` 值：

```
{f1 = v1, f2 = v2, ... , fn = vn}
```
* `fn` 为字段名，`vn` 为字段值

`record` 类型：

```
{f1: t1, f2: t2, ... , fn: tn}
```
* `tn` 为字段类型

构建 `record`：

```
{f1 = e1, f2 = e2, ... , fn = en}
```
* `en` 为表达式

访问元素：

```
#fn record
```
* `fn` 为字段名，`record` 为 `record` 类型的值。

> 注意：`record` 通过字段名访问，而 `tuple` 通过位置访问。通过名字还是位置访问，是一个广泛的设计问题，比如 ML Java 的方法采用了混合模式，调用者使用位置访问，而被调用者（函数内部）使用名字访问。

### `tuple` 是 `record` 的语法糖

```
val x = {2 = 20 + 1, 1 = 4}
```

在 REPL 中执行上面语句，发现 `x` 类型为 `int * int`，是 `pair` 类型，这是因为在 ML 中其实不存在 `pair` `tuple` 这样的东西，`tuple` 只不过是 `record` 的另一种简洁的写法而已，而 `tuple` 的 `type-check` 规则、`evaluate` 规则都可以用 `record` 解释。

`andalso` `orelse` 是 `if-then-else` 的语法糖。

## 使用 `datatype binding` 构建 `one of` 类型

ML 有 3 种绑定：

1. `variable binding`（使用 `val`）
2. `function binding`（使用 `fun`）
3. `datatype binding`（使用 `datatype`）

其中 `datatype binding` 是构建 `one of` 复合类型的非常好的方法。

```
datatype mytype = TwoInts of int * int
                | Str of string
                | Pizza

val a = TwoInts (1, 4);
val b = Str "Hi";
val c = Pizza;
```

* 定义 `mytype` 类型，该类型的值携带 `int * int` **or** `string` **or** nothing。
* `TwoIns` `Str` `Pizza` 都是 `constructor`，具备以下特点：
  1. `constructor` 要么是一个函数，类型为：`int * int -> mytype` `string -> mytype`
  2. 要么是一个 `mytype` 类型的值，比如 `Pizza`
* `abc` 都是 `mytype` 类型的值，分别为 `TwoInts (1, 4)` `Str "hi"` `Pizza`，注意，值携带了 `TwoInts` **标签**，表示该值是由 `TwoInts` 变体产生的。

> `mytype` 类型的值，如 `TwoInts (1, 4)` `Str "Hi"` `Pizza`，由两部分组成：
  1. `tag` 表示该值由哪个 `constructor` 构造产生
  2. `conrresponding data`

### `case` 表达式

要访问 `one of` 类型的数据，一般需要两个步骤：

1. 识别该值是由哪个构造函数产生（`null` `isSome`）
2. 获取对应的值（`hd` `tl` `valOf`）

对于 `datatype` 定义的类型值，我们使用 `case` 表达式 + 模式匹配完成这两个步骤。

```
fun f x =
  case x of
    Pizza => 3
  | TwoInts (a, b) => a + b
  | Str s => String.size s
```
1. 若 `x` 为 `TwoInts (1+2, 2+3)`，首先计算 `x`，得到值为 `TwoInts (3, 5)`
2. 值 `TwoInts (3, 5)` 匹配到 `TwoInts (a, b)` 分支，将 `a` 绑定到 3，将 `b` 绑定到 5
  * 注意这两个绑定的作用域仅仅是在 `a + b` 中有效的局部绑定（类似 `let`）
3. 计算 `=>` 右边的表达式，得到 8
4. 整个 `case` 表达式的计算结果为 8

> **注意**：`case` 表达式中的模式为 `constructor` + 正确数量的参数，这与表达式形式类似，但是两者概念不同，ML 不会计算模式，而是 **匹配** 模式。

#### `case` 表达式的优势

相对使用类似 `isSome` `valOf` 这样的方法提取数据，`case` 表达式有如下优势：

1. `case` 表达式 **不能遗漏** 任意 `constructor`
2. `case` 表达式 **不能重复** 任意 `constructor`
3. 不会出现忘记使用 `isSome` 判断的情况，可以确定每种模式的处理方式，不会出现 **模式混乱**
4. 模式匹配更加强大

### `type` 同义词

```
type another_name = t
```
* `t` 是已经存在的类型
* 创建类型 `t` 的同义词 `another_name`
* 二者在 **任何方面** 都可以 **替换使用**

例子：

```
datatype suit = Club | Diamond | Heart | Spade
datatype rank = Jack | Queen | King | Ace | Num of int
// card 是 suit * rank 的别名
type card = suit * rank
// name_record 是后面 record 类型的别名
type name_record = { student_num : int option,
                     first       : string,
                     middle      : string option,
                     last        : string }
```

到目前为止，`type` 同义词仅仅简化了代码书写，在模块化中会扮演更加重要的作用。

### `list` `option` 用于 `case` 表达式

```
fun inc_or_zero intoption =
  case intoption of
      NONE => 0
    | SOME i => i + 1;
```

* `NONE` `SOME` 是 `option` 的构造函数；
* 优先选用 `case` 表达式，不要用 `isSome` `valOf`；

```
fun sum_list xs =
  case xs of
      [] => 0
    | x :: xs' => x + sum_list xs';
```

* `[]` `::` 是 `list` 的构造函数；

> 虽然 `case` 表达式满足大部分场景，但 ML 还是提供了 `null` `hd` `tl` 等函数，原因：
1. `hd` `tl` 有时作为参数传递给其他函数；
2. 部分场景下，`hd` `tl` 更方便；
3. 即使 ML 没有内置这些函数，我们自己也能轻易实现，比如：
```
fun isNull x =
  case x of
    [] => true
  | x :: xs => false
```

### 泛型

在 ML 中，`list` 并非类型，`int list` 才是类型；`option` 并非类型，`int option` 才是。`list` `option` 是类型构造函数，必须添加 **类型参数** 才会生成真正的类型。

ML 中 `option` 定义如下：

```
datatype 'a option = NONE | SOME of 'a
```
* `option` 不是类型，`int option` 才是。

## 在 `each of` 类型中使用模式匹配

在 `one of` 类型中，判断使用哪个构造函数、提取元素，**只能** 使用模式匹配， 其实在 `each of` 类型中也可以使用。

### 在 `tuple` `record` 中使用模式匹配

* 模式 `(x1, ... , xn)` 可以匹配值 `(v1, ... , vn)`，并且创建 `local binding`，将 `x1` 绑定到 `v1` ...
* 模式 `{f1 = x1, ... , fn = xn}` 可以匹配值 `{f1 = v1, ... , fn = vn}`，并创建 `local binding`，将 `x1` 绑定到 `v1` ...
  + **注意**：字段名的顺序无关紧要

#### 最基本版

```
fun sum_triple (t: int * int * int) =
  case t of  (* 不良风格：case 表达式只有一个 case 分支！ *)
      (x, y, z) => x + y + z;

val result = sum_triple (1, 2, 3) = 6;

fun full_name (r: {first: string, middle: string, last: string}) =
  case r of
      {first = x, middle = y, last = z} => x ^ " " ^ y ^ " " ^ z;

val result = full_name {first = "Kyke", middle = "", last = "Song"};
```
* `case` 表达式只有一个分支，不良风格！

#### 改进：变量绑定 + 模式匹配

前面变量绑定讲到，可以将表达式 `e` 绑定到变量 `v` 上，如：`val v = e`，但其实 **变量也是模式**，所以该语法实际为：

```
val p = e
```

* `=` 左边的 `p` 其实是模式，当其为单个变量时，是用单个变量匹配 `e` 的计算结果；
* 当 `p` 为 **复杂模式** 时，将匹配 `e` 计算结果的结构，可以方便提取其元素值；

```
fun sum_triple2 t =
  let val (x, y, z) = t
  in
      x + y + z
  end;
fun full_name2 r =
  let val {first = x, middle = y, last = z} = r
  in
      x ^ " " ^ y ^ " " ^ z
  end;
```

#### 改进：函数绑定 + 模式匹配

函数绑定时的 **参数** 也是模式：

```
fun f p = e
```
* `p` 是模式，用来匹配调用时的 **实际参数**

```
fun sum_triple3 (x, y, z) =
  x + y + z;
fun full_name {first = x, middle = y, last = z} =
  x ^ " " ^ y ^ " " ^ z;
```
* `type-checker` 自动推断出 `sum_triple3` 类型为 `int * int * int -> int`。

#### 函数绑定 -> 参数只有一个

下面函数，参数类型是 3 个 `int` 呢，还是一个包含 3 个 `int` 的 `triple` 呢？

```
fun sum_triple (x, y, z) =
  x + y + z;
```

在 ML 中，函数 **只有一个参数**，多参函数其实是接受一个 `tuple` 参数的函数。

```
fun rotate_left (a, b, c) = (b, c, a)
// 可以连续调用
fun rotate_right t = rotate_left (rotate_right t)
```
* 在 Java 中，如果函数 **返回多个值**，则连续调用函数会很麻烦，因为需要给每个参数赋值；
* 在 ML 中，函数永远接受一个 `tuple` 参数，所以上面的 `rotate_left` 可以直接将函数返回值作为调用参数；

## ML 使用 **模式** 进行类型推断

```
fun sum_triple (x, y, z) =
  x + y + z;
```

* `type-checker` 根据 `(x, y, z)` 可以获取参数类型为包含 3 个元素的 `tuple`；
* `type-checker` 根据 `x + y + z` 可以判断元素为 `int` 类型；
* 综合以上两点推断出函数参数为 `int * int * int` 的 `tuple` 类型。

```
fun sum_triple t = (* t 需要指定类型 *)
  #1 t + #2 t + #3 t;
```
* `type-checker` 只能推断出参数 `t` 是 **至少** 包含 3 个 `int` 元素的 `tuple`，无法推断出其是否包含其他元素，所以 `t` 必须指定类型；

```
fun partial_sum (x, y, z) =
  x + z;
```
* 参数 `y` 没有使用，可以为任意类型 `'a`
* 该函数类型为 `int * 'a * int -> int`
* `partial_sum (1, 2, 3)` & `partial_sum (1, "ss", 3)` 都是合法的

### 泛型类型 & 相等类型

```
fun append (xs, ys) =
  case xs of
      [] => ys
    | x :: xs' => x :: append (xs', ys);
```
* 函数类型为 `'a list * 'a list -> 'a list`
* 用 `int` 替换 `'a` 函数类型变成 `int * int -> int`，说明 `'a list * 'a list -> 'a list` 更加通用

```
fun same_thing (x, y) =
  if x = y then "yes" else "no";
```
* 函数类型为 `''a * ''b -> string`
* `''a` `''b` 必须是 `Equality Type`

> ML 中 `=` 可以用于很多类型，比如 `int` `string` 包含可用 `=` 类型元素的 `tuple`；但不能用于 `function type` `real` 类型，ML 将能用 `=` 比较的类型称为 `Equality Type`，用 `''a` 表示。

### 嵌套模式

ML 中，模式可以 **任意嵌套**，将 **嵌套的 `case` 表达式** 转换为 **嵌套的模式**，代码更加清晰。

* 模式：描述了值的 **结构**
* 值：具有一定结构

> 模式匹配，就是拿模式去匹配值，若该值结构为模式所描述的，则利用模式 **提取** 值中组成元素。

```
exception ListLengthException;

(*  'a list * 'b list * 'c list -> ('a * 'b * 'c) list *)
fun zip3 list_triple =
  case list_triple of
      ([], [], []) => []
   |  (hd1 :: tl1, hd2 :: tl2, hd3 :: tl3) => (hd1, hd2, hd3) :: zip3 (tl1, tl2, tl3) (* noempty list *)
   | _ => raise ListLengthException;

val result = zip3 ([1, 2, 3 ], [4, 5, 6], [7, 8, 9]) = [(1,4,7),(2,5,8),(3,6,9)];

(* ('a * 'b * 'c) list -> 'a list * 'b list * 'c list *)
fun unzip3 tlist =
  case tlist of
      [] => ([], [], [])
    | (a, b, c) :: tl => let val (x, y, z) = unzip3 tl
                  			 in
                  			     (a :: x, b :: y, c :: z)
                  			 end;

val result = unzip3 [(1,4,7),(2,5,8),(3,6,9)] = ([1, 2, 3 ], [4, 5, 6], [7, 8, 9]);
```
* `zip3` 需要用 `_` 匹配其他有元素为 `empty list` 的情况
* `unzip3` 只有空 `list`、非空 `list` 两种情况
* `unzip3` 实现很巧妙

#### 嵌套模式例子

```
fun nondecreasing intlist =
  case intlist of
      x :: y :: tl => x <= y andalso nondecreasing (y :: tl)
    | _ => true;

val result = nondecreasing [1, 2, 3, 3, 4] = true;
val result = nondecreasing [1, 2, 2, 3, 2] = false;
```

```
datatype sign = P | N | Z;

fun multsign (x, y) =
  let fun sign x = if x = 0 then Z else if x > 0 then P else N
  in
      case (sign x, sign y) of
      	  (_, Z) => Z
      	| (Z, _) => Z
      	| (P, N) => N
      	| (N, P) => N
      	| _ => N
      (*	| (P, P) => P
      	| (N, N) => P *)
  end;
```
* 可以将结果为 `P` 的 `case` 省略，以 `_` 通配，但是这样 `type-checker` 就无法发现 **遗漏** 的 `case`

#### 风格

1. 避免使用嵌套的 `case` 表达式，用嵌套的 `pattern` 代替，比如 `unzip3` `nondecreasing`
2. 使用 `tuple` 模式匹配，比如 `zip3` `multsign`
3. 不需要提取元素时，使用 `_`，比如 `multsign`

### 模式匹配精确定义

若有模式 `p` 和值 `v`，模式匹配会：

1. 检查 `p` 是否能匹配 `v`
2. 若能匹配，则引入一些 **绑定**

> 注意：`v` 可以是 `case` `of` 之间表达式的计算结果，可以是 `val` 绑定邮编表达式的计算结果，可以是函数调用者使用的实际参数。

模式可以嵌套，因此其定义是 **递归** 的，对于不同模式有不同匹配规则：

1. 若模式 `p` 是 **变量 `x`**，则匹配成功，且引入绑定：`x` 绑定到值 `v`；
2. 若模式 `p` 是 `_`，则匹配成功，但不引入任何绑定；
3. 若模式 `p` 是 `C p1`，只有当值为 `C v1`，且 `p1` 能够匹配 `v1` 时才能匹配；
  * 相同的构造函数 `C`
  * `p1` 能匹配 `v1`
4. 若模式 `p` 是 `(p1, ... , pn)`，值为 `(v1, ... , vn)`，**当且仅当** `p1` 匹配 `v1`，`p2` 匹配 `v2` ... 时，整个模式 `p` 才匹配成功，并且引入 n 个绑定，分别为：`p1` 绑定到 `v1`，`p2` 绑定到 `v2` ...

### Function Pattern

另一种模式使用场景：

```
fun append ([], ys) = ys
  | append (x :: xs, ys) = x :: append (xs, ys);

val result = append ([], [1,2,3]) = [1,2,3];
val result = append ([1,2,3], [4,5,6]) = [1,2,3,4,5,6];
```

```
fun f x =
  case x of
    p1 => e1
  | p2 => e2
  ...  
```

可以改写为：

```
fun f p1 = e1
  | f p2 = e2
  ...
```

## 异常

### 异常绑定

异常绑定，通过 `exception` 关键字引入异常：

```
// 定义异常
exception MyException
exception AnotherException of int * int
// 抛出异常
raise MyException
raise AnotherException (2, 4)
```

### 基本用法

```
exception MyEmptyException;

fun maxList xs =
  case xs of
      [] => raise MyEmptyException
    | x :: [] => x
    | x :: xs' => Int.max(x, maxList xs');

val result = maxList [];

fun maxList2 (xs, ex) =
  case xs of
      [] => raise ex
    | x :: [] => x
    | x :: xs' => Int.max(x, maxList2(xs', ex));

val result = maxList2 ([], MyEmptyException);
```
* `maxList2` 在 `xs` 为空列表时，抛出 `ex` 异常；

### 异常处理

```
e1 handle ex => e2
```

1. 若 `e1` 抛出异常，则该异常用于 `ex` 进行模式匹配
2. 若匹配成功，则执行 `e2`
  * `e2` 抛出异常，则继续向上传播
3. 若匹配失败，将继续向上传播

### 异常的本质

异常实际为:

1. 创建 `exn` 类型值的 **构造函数**
2. 或者，为 `exn` 类型的值

## 尾递归

递归经常被诟病为 **低效**，原因如下：

1. 每个函数调用对应一个 **调用栈帧**（保存函数本地变量、执行到哪里等信息）；
2. 若函数执行结束，则弹出栈帧，若 **未执行结束**，则不弹出；
3. 递归调用，很容易产生成百上千的 **栈帧**；

```
fun fact n =
  if n = 0
  then 1
  else n * fact (n-1);
```

调用 `fact 3` 的执行栈帧最大多有 4 个，分别保存 `fact 3` `fact 2` `fact 1` `fact 0`。

改用尾递归实现之后：

```
fun fact n =
  let fun aux (n, acc) =
        if n = 0
        then acc
        else aux (n - 1, acc * n)
  in
    aux (n, 1)
  end;
```

* 用尾递归（尾调用）优化之后，只需要一个栈帧；
* `aux` 是尾递归函数，即：递归调用在函数最后，且除了调用 `aux` 外，没有任何其他操作
  1. `3 * aux` 不是尾递归
  2. `aux` 调用之后，若有其他计算，则不是尾递归
* `fact` 对 `aux` 的调用为 **尾调用**

### 编译器对尾递归的优化

ML 编译器对尾递归会 **特殊处理**，当识别到尾递归后，编译器会：

1. **弹出** `caller` 的调用栈，使 `callee` 可以复用 `caller` 的栈空间；
2. 通过优化，尾递归与循环一样高效；

### 普通递归函数 -> 尾递归函数

这种转换有个基本套路：

1. 定义 `local helper function`，接受 `accumulator` 作为参数；
2. 将原本函数的 `base case` 作为 `accumulator` 的 **初始值**；
3. `final accumulator` 即为 **最终结果**；

非递归版本：

```
fun reverse1 xs =
  case xs of
      [] => []
    | x :: xs' => reverse1 (xs') @ [x];
```
* `base case` 为 `[]`

递归版本：

```
fun reverse xs =
  let fun f (xs, acc) =
      	case xs of
      	    [] => acc
      	  | x :: xs' => f (xs', x :: acc)
  in
      f (xs, [])
  end;
```
* `base case` 变为 `f` 的 `accumulator` 参数的初始值；
* `accumulator` 最终值即为最终结果；

### 尾递归并非无所不能

尾递归能大大提升效率，但是 **并非所有函数** 都能转换为尾递归版本，比如处理 `tree` 的函数，最多只能将其中一条递归函数转换为递归版本。

不要过早优化，代码的 **可读性** 仍然很重要。

### 尾递归定义

`tail call`：若函数调用发生在 `tail position`，则该函数为 **尾调用**。

1. 若表达式 `S` 不在 `tail position`，则其所有子表达式也不在 `tail position`
2. 函数定义 `fun f p = e` 中，`e` 为 `tail position`
3. 若 `if e1 then e2 else e3` 本身处于 `tail position`，则 `e2` `e3` 为 `tail position`（`case` 表达式类似）
4. 若 `let b1 ... bn in e end` 本身处于 `tail position`，则 `e` 为 `tail position`
5. 函数调用参数 `e` 不是 `tail position`（因为 `e` 表达式计算完成后，还需要进行函数调用）

> 上述定义是递归的。
