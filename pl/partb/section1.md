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

















##
