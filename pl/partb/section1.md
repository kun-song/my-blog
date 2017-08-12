# 第一部分：基础

## Racket 简介

* Racket 与 ML 类似，具有：匿名函数、闭包、一切皆为表达式、无 return 语句等；
* 语法简介，但有很多高级特性，比如 `macros`, `modules`, `quoting/eval`, `continuations`, `contracts`；
* 但本课程中不会使用 **模式匹配**；

```
; 单行注释
#lang racket

; Racket 中，一个文件即为一个模块，模块内容默认 private，为方便测试，添加如下几行
(provide (all-defined-out)) ; 注释

(define s "Hello World!")
```

* DrRacket 支持多种语言，所以需要在程序开始指定所需语言：`#lang racket`；
* 文件即模块；

> 与 ML 程序是绑定的集合（`a collection of val-bindings`）类似，Racket 程序是定义的集合（`a collection of definitions`）。

Racket 中，`+` 为函数，下面 `(+ x 2)` 为函数调用：

```
(define x 3)
(define y (+ x 2))
```

### 函数定义

Racket 使用 `lambda` 定义匿名函数，其语法为 `lambda (x) e`

```
(define cube
  (lambda (x)
    (* x x x)))
```

使用 `lambda` 需要多打字，所以 Racket 提供了函数定义的 **语法糖**：

```
(define (cube x)
  (* x x x))
```

> ML 中，匿名函数 `fn` 是函数定义 `fun` 的语法糖，且匿名函数不能用于递归；在 Racket 中相反，函数定义是匿名函数定义的语法糖，且两者都可以用于递归。

Racket 条件表达式为 `if e1 e2 e3`：

```
(define (pow1 x y)
  (if (= y 0)
      1
      (* x (pow1 x (- y 1)))))
```

> Racket 的条件表达式比较特殊，规则为先计算 `e1`，若其为 `#f`，则结果为 `e3`，其他任何值（包括 `#t`），结果为 `e2`。

### 柯里化（本质：返回函数）

Racket 的函数也是闭包，而柯里化只是闭包的一种惯用法，所以 Racket 可以轻易实现闭包：

```
(define (pow1 x y)
  (if (= y 0)
      1
      (* x (pow1 x (- y 1)))))

(define pow2
  (lambda (x)       ; 返回函数
    (lambda (y)     ; 返回函数
      (pow1 x y))))

(define three-to-nth (pow2 3))
(three-to-nth 2)
```

Racket 对柯里化函数 **调用** 无任何语法糖，所以对于 `pow1` `pow2` 的调用，分别如下：

```
(pow1 2 3)
((pow2 2) 3)
```

* `(pow2 2)` 计算结果为接受一个参数的函数，用 `3` 继续调用它；
* `(pow1 2 3)` 中，`pow1` 为接受两个参数的函数，如果用类似形式调用 `pow2`，则参数个数错误；

Racket 对于柯里化函数的 **定义** 有语法糖：

```
(define ((pow x) y)
  (if (= y 0)
      1
      (* x ((pow x) (- y 1)))))
```

#### 注意

因为 Racket 的多参函数，本质是真正的多参，而非语法糖（ML 的多参为语法糖，实际只有一个 `tuple` 参数），因此柯里化在 Racket 中并不常用。

### List

Racket 与 ML 的列表非常相似，有着相似的处理函数、相似的惯用法：

| 操作            | Racket         |       SML     |
| :------------- | :------------- | :------------- |
|       空表      |   `null`       |       []       |
|  Cons 操作      |   `cons`       |      `::`      |
|  head of list  |    `car`       |      `hd`      |
| tail list      |    `cdr`       |      `tl`      |
|       创建      | `(list 1 2 )`  |   `[1, 2, 3]`  |
|        判空     |   `null?`      |     `null`     |

接下来会实现一系列 `list` 相关的经典函数，其中 `map` `filter` `append` 是 Racket 语言内置特性，所以定义同名函数时会 `shadow` 内置版本。

#### 列表求和

与 ML 版本非常类似：

```
; 列表求和
(define sum
  (lambda (xs)
    (if (null? xs)
        0
        (+ (car xs) (sum (cdr xs))))))
```

使用语法糖后，与 ML 版本更加类似：

```
(define (sum2 xs)
  (if (null? xs)
      0
      (+ (car xs) (sum2 (cdr xs)))))
```

#### `map` 函数

```
(define map
  (lambda (f xs)
    (if (null? xs)
        null
        (cons (f (car xs)) (map f (cdr xs))))))
```

使用语法糖：

```
(define (map2 f xs)
  (if (null? xs)
      null
      (cons (f (car xs)) (map2 f (cdr xs)))))
```

#### `filter` 函数

```
(define my-filter
  (lambda (f xs)
    (if (null? xs)
        null
        (if (f (car xs))
            (cons (car xs) (my-filter f (cdr xs)))
            (my-filter f (cdr xs))))))
```

语法糖版本：

```
(define (my-filter2 f xs)
  (if (null? xs)
      null
      (if (f (car xs))
          (cons (car xs) (my-filter2 f (cdr xs)))
          (my-filter2 f (cdr xs)))))
```

#### `append` 函数

```
(define append
  (lambda (xs ys)
    (if (null? xs)
        ys
        (cons (car xs) (append (cdr xs) ys)))))
```

语法糖版本：

```
(define (append2 xs ys)
  (if (null? xs)
      ys
      (cons (car xs) (append2 (cdr xs) ys))))
```

## 语法与括号

Racket 的语法非常简单，Racket 程序组成可以归类为两部分：

1. Some form of `atom`
  * #t, #f, 34, "hi", null
  * 标识符：
    + 变量
    + a special form： `define`, `lambda`, `if`...（标识了语法内置特性）
2. A `sequence` of things in parentheses (t1 t2 ... tn).

> 对于序列而言，首元素决定了序列的语义：
1. 若首元素为 `special form`，则其语义由该 `special form` 确定（每个 `special form` 都有不同语义）。比如 `define`，则剩余部分要么是
  * 变量定义（包含真正的变量 + 函数）
  * 语法糖版本的函数定义
2. 若首元素不是 `special form`，则其为 **函数调用**，比如 `(e)` 首先会计算 `e` 表达式，将其结果作为 **函数**，并用 0 个参数调用它，所以 `(42)` 将会得到语法错误，因为 42 不是函数。

Racket 的语法使其非常容易转化为 **语法树**，也无需考虑各种操作符的 **优先级**，使 Racket 程序没有任何 **歧义**。

## 动态类型（初探）

* 优点：非常灵活，可以使用非常灵活的数据结构
  1. 比如 `(list 1 2 (list 3 4))` 在 Racket 中可以直接使用，而 ML 中列表的元素类型必须相同，所以要实现前面的形式，需要定义 `datatype` 才能使用；
* 缺点：错误可能在运行时才能发现（有的运行场景少见，可能很长时间不发现，突然出问题）
  1. 比如 `(if (= x 2) 2 (3 + 4)` 在 `x = 2` 时并无运行时错误，其他情况下都会报错；

### 嵌套的列表、数字求和

假设列表 `xs` 可以包含其他列表、数字，比如 `(list 1 2 (list 3 4 (list 5 6)) (list 32 0))`，使用函数 `sum` 对其求和。

```
(define (sum1 xs)
  (if (null? xs)
      0
      (if (number? (car xs))
          (+ (car xs) (sum1 (cdr xs)))
          (+ (sum1 (car xs)) (sum1 (cdr xs))))))
```
* 首先判断是否为空列表，若为非空列表，则判断其首元素是否为列表；
* `(sum1 (list 1 "Hi"))` 将会报错，因为 `Hi` 是字符串，不能用于 `+` 函数；

若想能够处理列表中出现的非数字类型，比如直接将其忽略，可以这样写：

```
(define (sum2 xs)
  (if (null? xs)
      0
      (if (number? (car xs))
          (+ (car xs) (sum2 (cdr xs)))
          (if (list? (car xs))
              (+ (sum2 (car xs)) (sum2 (cdr xs)))
              (sum2 (cdr xs))))))

(sum2 (list 1 2 (list 3 4 "Hi" (list 5)) 10))
```
* 增加一个 `if` 判断；

## `Cond`

`cond` 用于取代 **嵌套的 `if`** 表达式，语法如下：

```
(cond [e1a e1b]
      [e2a e2b]
      [e3a e3b]
      ...
      [eNa eNb])
```

* `eXa` 是 `test`，若 `test` 为 `#f`，则跳过该分支，其他情况（包括 `#t`）则计算 `eXb`；
* 最后一个分支，一般 `test` 为 `#t`，即 `[#t e]`，类似其他语言中的 `default` 分支；
* `cond` 有点类似 Java 的 `switch`；

```
; 使用 cond 改写 sum1
(define (sum3 xs)
  (cond [(null? xs) 0]
        [(number? (car xs)) (+ (car xs) (sum3 (cdr xs)))]
        [#t (+ (sum3 (car xs)) (sum3 (cdr xs)))]))

; 使用 cond 改写 sum2
(define (sum4 xs)
  (cond [(null? xs) 0]
        [(number? (car xs)) (+ (car xs) (sum4 (cdr xs)))]
        [(list? (car xs)) (+ (sum4 (car xs)) (sum4 (cdr xs)))]
        [#t (sum4 (cdr xs))]))
```

### 关于 `true` 的处理

在 Racket 中，`if` `cond` 表达式中的 `test expression` 可以 `evaluate to anything`，而结果的真假按照如下规则判断：

* Treat anything other than `#f` as true;

该特性在静态语言中 **无任何意义**，因为静态语言的 `test expression` 结果必须为 `bool` 类型（有的语言中，其他类型可以转换到布尔类型，但这是另外一个维度的问题）。

```
; 14
if (10 14 15)
; 14
if (null 14 15)
; 15
if (#f 14 15)
```

* 注意，有的语言中，将空表、空字符串之类视为 `false`，但 Racket 将其视为 `true`；

## Local bindings

ML 使用 `let` 表达式进行局部绑定，在 Racket 中，有 4 中创建局部绑定的方式：

1. `let`
2. `let*`
3. `letrec`
4. `define`

这 4 种方式 **语义不同**，适用于 **不同场景**，若任何一个都可以满足，则优先使用 `let`。

### `let`

`let` 可以绑定 **任意数量** 的局部变量。

The expressions are all evaluated in the environment from `before the let-expression`:

* Except the `body` can use `all` the local variables of course；
* 与 ML `let` 不同；
* 适用于 `(let ([x y][y x]) …)`；

```
(define (silly-double x)
  (let ([x (+ x 2)]
        [y (+ x 3)])
    (+ x y -5)))
```
* `(+ x 2)` `(+ x 3)` 中的 `x` 引用 `(silly-double x)` 中的 `x`；

### `let*`

The expressions are evaluated in the environment produced from `the previous bindings`.

与 ML 的 `let` 基本一致。

```
(define (silly-double2 x)
  (let* ([x (+ x 3)]
         [y (+ x 2)])
    (+ x y -8)))
```

### `letrec`

The expressions are evaluated in the environment that includes `all the bindings`.

```
(define (silly-triple x)
  (letrec ([y (+ x 2)] ;    若改为 [y (+ w 2)] 则报错，因为此处 w 还未初始化
           [f (lambda (z) (+ z w y x))]
           [w (+ x 7)])
    (f -9)))
```
* `letrec` 一般用于 `mutual recursion`；
* But expressions are still **evaluated in order**: accessing an `uninitialized` binding produces an error

`letrec` 非常适合定义 `mutual recursion`:

```
(define (silly-mod2 x)
 (letrec ([even? (lambda (x)(if (zero? x) #t (odd? (- x 1))))]
          [odd? (lambda (x)(if (zero? x) #f (even? (- x 1))))])
   (if (even? x) 0 1)))
```

### local definition

```
(define (silly-mod2 x)
 (define (even? x)(if (zero? x) #t (odd? (- x 1))))
 (define (odd? x) (if (zero? x) #f (even?(- x 1))))
 (if (even? x) 0 1))
```

local definition 可用于函数定义开始位置，若使用 local definition 定义 **局部变量**，则其 **语义** 与 `letrec` 相同。

## Top level definitions

在 Racket 中，**文件即模块**，模块由 `a sequence of definitions` 组成。

* ML 的模块作用域类似一个 `let*`：只能引用 **前面** 定义好的绑定；
* Racket 的模块作用域类似一个 `letrec`：可以 **前后** 引用；

`letrec` 的优点是绑定/定义可以在 **任意位置**，不必像 ML 中，`mutual recursive` 函数必须用 **特殊语法** 放在一起；缺点是有几条限制需要遵守：

1. Refer to **later bindings** only in **function bodies**
  * 因为虽然绑定定义可以 **任意顺序**，但依然按照从前到后的顺序 **计算、初始化**；
  * 使用 **未初始化** 的绑定，出现错误；
2. Unlike ML, cannot define the same variable **twice** in module
  * 因为可以前后引用，所以无法确认究竟使用哪个

```
; 可以引用后面定义的 b
(define (f x) (+ x b))

(define b 3)

; 可以应用前面定义的 b
(define c (+ b 4))

; 不能在 e 未初始化时使用
;(define d (+ e 3))

(define e 5)

; 不能重复定义 f
;(define f 17)
```

**注意**：

* 模块 A 可以引入模块 B 的内容，此时在 A 中可以重复定义 B 的绑定，从而 `shadow` B 中的相同定义；
* 本节的 toplevel 不是真正意义上的 toplevel，实际上每个文件是一个单独的 `letrec`，文件之间是可以 shadow 的；

## `set!` 赋值操作符

Racket 拥有真正的赋值操作符：`set!`，其作用与 Java 中的赋值完全相同，需要 **非常谨慎** 使用。

> ML 无真正的赋值操作，ML 的重新绑定实质为 `shadow`，并未改变之前的绑定。Racket 的赋值将影响 **所有** 使用该变量的地方。

```
; Racket 有与 Java 等作用完全性相同的赋值操作符
(define b 3)

; f 为闭包，引用 b
(define f (lambda (x) (+ x b)))

; 计算 （+ b 4)，结果为 7，所以 c 为 7
(define c (+ b 4))

; 对 b 重新赋值，所有用到 b 的地方都会被影响
(set! b 5)

; z = 4 + 5 = 9
(define z (f 4))

; w = c = 7
(define w c)
```

* 该例明显看出 Racket 的赋值与 SML 的不同，`(set! b 5)` 之后，闭包 `f` 也被影响了，其引用的 `b` 也变成了 5；
* 在 SML 中，`f` 引用的 `b` 在重新赋值后，依然保持原本的值 3，所以 SML 的赋值为 `shadow`；

### 使用副本消除 `set!` 带来的问题

`set!` 将引发各种各样的问题，对于可能会改变的变量，使用 **副本** 以避免受到影响，最终得到的效果与 ML 相同：

```
(define b 3)
(define f2
  (let ([b b])
    (lambda (x) (+ x b))))
```

进一步，可能有人会重新定义 `+` `*` 等函数，那是否需要将此类函数也做个副本呢？比如：

```
(define b 3)
(define f2
  (let ([b b]
        [* *]
        [+ +])
    (lambda (x) (+ x b))))
```

在 Scheme 中确实可能需要，因为 Scheme 允许修改 `+` 等的定义，但在 Racket 中则无必要，因为：

1. Racket 的每个文件即为一个模块；
2. 若一个模块中，对于 toplevel 变量未使用 `set!`，则在 **其他模块** 中禁止使用 `set!`，否则报错；
3. Racket 中的 `+` `*` 定义所在模块，确实未使用 `set!`，因为其他模块无法修改它们；

实际的 Racket 代码不推荐使用 `set!`，本节的例子只是为了展示使用 **副本** 来避免修改导致的错误。

### `begin`

Racket 的 `begin` 语法用于定义一系列有 **副作用** 的定义：

```
(begin e1 e2 ... en)
```

* `begin` 表达式的结果为最后一个 `en` 的结果；
* `e1 -> en-1` 用于产生 **副作用**；

```
(define x (begin
  (+ 5 3)
  (* 3 8)
  (- 3 2)
))
```

## `cons` 的真相

之前我们使用 `cons` 构建列表，但实际上，`cons` 构建的是 `pair`，而列表只是 **嵌套的 `pair`**，最后以 `null` 结尾。

`cons` 构建的 `pair` 被称为 `cons cell`，是不可变的。

```
; cons 构建的，实际为 pair，而并非列表
(define pr (cons 12 (cons #t "Hi")))

; cons 最后一个元素，若以 null 结尾，则为 list
(define lst (cons 12 (cons #t (cons "Hi" null))))
```

* `pr` 为 `'(12 #t . "Hi")`
* `lst` 为 `'(12 #t "Hi")`

可以看到 `pr` 打印出的内容，与列表非常像，但中间多了一个 `.`；而当 `cons` 最后一个元素为 `null` 时，构建出的 `pair` 就是 `list`。

> `list` 是特殊的嵌套 `pair`。

使用 `car` `cdr` 获取 `cons cell` 元素

前面将 `car` `cdr` 用于列表，因为 `list` 是特殊的、嵌套的 `pair`，所以必然可以用于 `pair`：

```
; 结果为 "Hi"
(cdr (cdr pr))
; 结果为列表 ("Hi")
(cdr (cdr lst))
```

使用 `list?` `pair?` 分别判断 `list` 和 `pair`：

```
(list? lst)
(pair? pr)
; list 是特殊的 pair
(pair? lst)
```

* `list?` 对与 `proper list`、`null` 返回 #t
* `pair?` 对于 **任意** 用 `cons` 构建的东西返回 #t
  + 注意：`pair?` 对于空表 `null` 返回 #f

### `proper list` vs `improver list`

`proper list` 即为列表，而 `improver list` 为使用 `cons` 构建的，则最后不以 `null` 结尾的 `pair`，Racket 的部分内置函数，只能用于 `proper list`，比如 `length`：

```
; 报错
(length pr)
; 3
(length lst)
```











##
