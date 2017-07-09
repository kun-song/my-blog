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











































噼噼啪啪铺
