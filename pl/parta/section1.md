# 第一部分

## syntax & semantics

`ML` 程序是 `bindings` 序列，每个 `binding` 需要两步处理：

1. `type-check`
  * `type-check` 根据 `static environment` 决定当前 `binding` 的类型。
  * `static environment` 可以认为由 **前面** 所有绑定的 **类型** 组成。
2. `evaluate`
  * `binding` 的计算根据 `dynamic environment` 得出。
  * `dynamic environment` 可以认为由 **preceding bindings** 的 **值** 组成。

有时用 `context` 作为 `static environment` 的同义词，而单称 `environment` 一般指 `dynamic environment`。

变量绑定 `syntax` 如下：

```
val x = e;
```

* `x` 为任意变量，`e` 为任意表达式。
* `;` 在文件中可以，在 `sml` 命令行中必选，提示解释器绑定到此结束。

以上只是 `syntax`，还需要知道绑定的 `semantics`，即：**如何 `type-check` + 如何 `evaluate`** 绑定。

* `type-check`
  1. 使用 `current static environment` 对 `e` 进行类型检查
  2. 产生 `new static environment`（= `current static environment` + `e` 的类型）
* `evaluate`
  1. 使用 `current dynamic environment` 对 `e` 进行计算
  2. 产生 `new dynamic environment`（= `current dynamic environment` + `e` 的值）

`value` 也是 `expression` 的一种，是没有任何计算、不能继续化简的表达式，比如 7 就是 `value`，而 3 + 4 不是 `value`。

## 表达式

表达式嵌套深度可以无限深，对于一个表达式，需要从三个方面进行描述：

* `syntax` 描述表达式的形式
* `semantics` 描述表达式的 **含义**，包括两部分：
  1. `type-check` 规则
    * 产生一个 `type`
    * 失败，打印失败提示
  2. `evaluate` 规则
    * 产生一个 `value`
    * 抛出异常
    * 死循环

### `variable` 表达式

* `syntax`：字母、数字、下划线序列组成，不能以数字开头。
* `type-check` 规则
  1. 在 `static environment` 中查找该变量，若找到，则以其类型作为该变量类型；若找不到，`fail`。
* `evaluate` 规则
  1. 在 `dynamic environment` 中查找该变量，以其值作为计算结果（必然存在，因为已经通过 `type-check` 确定其存在）。

### 加法表达式

* 语法
  1. `e1 + e2`，其中 `e1` `e2` 都是表达式
* `type-check`
  1. 查找 `e1` `e2` 的类型，若两者都是 `int`，则 `e1 + e2` 也是 `int`
* `evaluate`（此时，已经确认 `e1` `e2` 都是 `int`）
  1. 计算 `e1` 得到 `v1`，计算 `e2` 得到 `v2`，则 `e1 + e2` 计算结果为 `v1` `v2` 的和

### `value`

* 所有 `value` 都是表达式，但并非所有表达式都是 `value`
* **Every value 'evaluates to itself' in zero steps**


### `if` 表达式

* syntax
  1. `if e1 then e2 else e3`
  2. 其中，`if` `then` `else` 都是 `ML` 关键字，`e1` `e2` `e3` 都是表达式
* type-check
  1. `e1` 必须是 `bool` 类型
  2. `e2` `e2` 可以是任意类型，但必须是 **相同类型** `t`
  3. 整个表达式的类型为 `t`
* evaluate
  1. 若 `e1` 为 `true`，则整个表达式的值为 `e2` 的值
  2. 若 `e2` 为 `false`，则为 `e3` 的值


## 多次绑定

在 `ML` 中没有赋值语句，只有绑定，但绑定可以进行多次：

```
val a = 10;
val b = a * 2;

// 再次绑定 a -> 5
val a = 5;
val c = a;
```

上述例子并没有修改 `a` 的值，而是将 `a` 重新绑定，以前 `a -> 10` 的绑定被 `shadowed`，重复绑定是非常差的风格，但清楚展示了 `environment` 的概念。

## 函数绑定

`ML` 程序是 `binding` 序列，除了 `variable binding` 外，还有 `function binding`。

### 语法：

```
fun x0 (x1: t1, ... , xn: tn) = e
```

### `type-check`

先 `type-check` 表达式 `e`，`type-check` 环境为

1. 以前的 `static environment`
2. 函数参数 `x1: t1, ... , xn: tn`
3. `x0 : (t1 * ... * tn) -> t`（用于递归）

若 `type-check` 结果为 `t`，则将函数绑定类型 `x0: (t1 * ... tn) -> t` 添加到 `static environment` 中。

### `evaluation`

* `A function is a value`，所以不会计算函数体
* 仅将 `x0` 加入 `dynamic environment`，后续表达式可以使用 `x0`

### 例子

```
fun pow(x: int, y: int) =
  if y = 0
  then 1
  else x * pow(x, y-1);
```

* 在函数体内，可以获取 **函数 pow**，**函数参数**
* 函数类型为 `(int * int) -> int`，其中 `*` 并非乘法，恰巧与乘法符号相同
* **绑定顺序** 非常重要，不能引用 **后面绑定** 的函数
* 函数参数数量 **不可变**

## 函数调用

### 语法

```
e0 (e1, ... ,en)
```
* 只有一个参数时，括号可以省略

### `type-check` 规则

若：

* `e0` 类型为 `(t1 * ... * tn) -> t`
* `e1 ... en` 类型分别为 `t1 ... tn`

则：

*  `e0 (e1, ..., en)` 类型为 `t`

### `evaluation` 规则

1. 在当前 `dynamic environment` 下，计算 `e0` 为 `fun x0 (x1: t1, ... , xn: tn) = e` 的函数。（因为函数仅仅是个 `value`，所以实际上仅仅在 `dynamic environment` 中查找名为 `e0` 的函数即可）
2. 在当前 `dynamic environment` 下，将参数 `e1 ... en` 计算为 `v1 ... vn`
3. 在 **函数定义时** `dynamic environment` + `x0`（递归） + `x1 -> v1 ...` 的环境中，计算表达式 `e`，其结果为函数调用结果。

## 数据结构

### `pairs`

#### 语法

```
(e1, e2)
```

#### `type-check` 规则

若 `e1` 类型为 `ta`，`e2` 类型为 `tb`，则 `(e1, e2)` 类型为 `(ta, tb)`（一种新类型）。

#### `evaluation` 规则

若 `e1` 计算结果为 `v1`，`e2` 计算结果为 `v2`，则 `(e1, e2)` 计算结果为 `(v1, v2)`。

### `list`

`pairs` 或 `tuple` 与 `list` 对比：

* `tuple` 元素数量确定、元素类型可以不同。
* `list` 元素数量不确定，类型必须相同。

`list` 的构造方式有两种：

1. 直接构造：`[1, 2, 3]`
2. 通过 `cons` 操作：`e1 :: list`

`list` 操作方法有三种：

1. `null` 函数：判断 `list` 是否为 `[]`
2. `hd` 获取头元素，若列表为 `[]`，抛出异常
3. `tl` 获取尾列表，若列表为 `[]`，抛出异常

#### 例子

```
fun sum_list(xs: int list) =
  if null xs
  then 0
  else hd xs + sum_list(tl xs)
```

操作 `list` 的函数基本都是 **递归** 函数，考虑两个方面：

1. 如何处理 `empty list`
2. 使用 `the tail of the list` 计算出 `non-empty list` 的结果

### `let` 表达式

定义 `local bindings`，作用有二：

* `for style and convenience`
* `for efficiency`（`local function bindings`）

#### 语法

```
let b1 b2 ... bn in e end
```
* `bi` 为任意绑定，`e` 为任意表达式

#### 类型检查规则

对 `bi` `e` 进行类型检查，整个 `let` 表达式的类型为 `e` 的类型（`e` 的类型可能需要 `bi` 的类型，所以 `bi` 的类型也是需要的）。

#### 计算规则

计算 `bi` `e`，`let` 表达式的计算结果为 `e` 的结果。

#### 闭包

```
fun count_from1_better(x: int) =
  let
    fun count(from: int) =
      if from = x
      then []
      else from :: count(from + 1)
  in
    count 1
  end;
```

局部函数绑定可以使用当前 `environment` 中的其他绑定，即：

* `count` 函数可以使用外部参数 `x`
* `count` 函数也可以使用 `let` 中前面定义的绑定

这里嵌套定义了 `count` 辅助函数，此类函数具有以下特点：

1. 不会用于其他地方
2. 若在其他地方使用，可能会被 **误用**，比如 `count` 若 `from` 传入值大于 `x`，则永远不会停止

#### 使用 `let` 表达式提升效率

```
(* [5, 4, 3, 2, 1] *)
fun countdown(from: int, to: int) =
  if from = to
  then to :: []
  else from :: countdown(from - 1, to);

(* [1, 2, 3, 4, 5] *)
fun countup(from: int, to: int) =
  if from = to
  then to :: []
  else from :: countup(from + 1, to);

(*
   1. bad_max 在 list 为递减列表时，效率很高，为 O(n)
   2. bad_max 在 list 为递增列表时，效率迅速下降，为 O(2^n)
*)
fun bad_max(xs: int list) =
  if null xs
  then 0 (* 坏风格，以后修正 *)
  else
      if hd xs > bad_max(tl xs) (* 1 *)
      then hd xs
      else bad_max(tl xs); (* 2 *)

(*
   1. 通过 let 表达式定义局部绑定，减少重复递归调用，时间复杂度提升为 O(n)
*)
fun better_max(xs: int list) =
  if null xs
  then 0
  else
      let val tl_ans = better_max(tl xs)
      in
    	  if hd xs > tl_ans
    	  then hd xs
    	  else tl_ans
      end;
```

#### 通过 `Option` 提升可读性

上面的例子里，对于空列表，返回了 0，但 0 并非空表中的最大元素，这不符合语义，可以使用 `Option` 改善。

##### `Option`

`Option` 类型为 `t option`，可以通过两种方式构建：

1. `NONE` 为 `'a option` 类型的值
2. `SOME e`，若 `e` 类型为 `t`，则 `SOME e` 类型为 `t option`

访问 `Option` 的值：

1. `isSome` 函数：`t option -> bool`
2. `valOf` 函数：`t option -> t`

```
fun better_max(xs: int list) =
  if null xs
  then NONE (* option *)
  else
      let fun max(xs: int list) =
        if null(tl xs)
        then hd xs
        else
          let val tl_ans = max(tl xs)
          in
        	  if hd xs > tl_ans
        	  then hd xs
        	  else tl_ans
          end;
      in
        SOME(max xs) (* option *)
      end;
```

### 逻辑运算符

```
e1 andalso e2
e1 orelse e2

not e
```

* `andalso` `orelse` 是语言关键字，因为它们不一定会计算 `e2`，而函数首先会计算所有参数。
* `not` 是函数。
* `e1` `e2` 可以是任意表达式，但计算结果 **必须是 `bool`**。

### 比较运算符

```
= <>    > < >= <=
```
* 6 个运算符最初是用于 `int`
* 后 4 个也可以用于 `real`（浮点型），但不能用于 `real` 和 `int` 比较
* 前 2 个可用于 **任意 `equality type`**, 但 `real` 不是 `equality type`，不能使用

## 不可变数据

```
fun sort_pair(pr: int * int) =
  if #1 pr < #2 pr
  then (#1 pr, #2 pr)
  else (#2 pr, #1 pr)

fun sort_pair(pr: int * int) =
  if #1 pr < #2 pr
  then pr
  else (#2 pr, #1 pr)
```

上面两种实现，在 `ML` 中 **无法区分**，因为 `pair` 是不可变的，所以返回参数的 `pr` 与返回一个 **`pr` 副本** 并没有区别：都没有办法修改返回值，而且也没有办法修改参数值。

但在 Java 中，这两种差别很大。

在 ML 中，经常使用 `alias`，即直接返回 `pr` 这种，而不需要使用副本，而 Java 为了避免数据被修改，经常需要返回副本。

## 学习语言的 5 个方面

1. 语法：怎么写
2. 语义：代码的含义
3. 习惯用法：该语言的常用表达方式
4. 库：语言携带、第三方
5. 工具：调试、格式化、REPL 等
