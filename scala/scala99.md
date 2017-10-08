# Scala 99

## List 相关

###01 Find the last element of a list

Example:

```
scala> last(List(1, 1, 2, 3, 5, 8))
res0: Int = 8
```

Scala 内置了求 `Seq` 最后一个元素的方法，所以最简单的方法就是直接调用 `Seq.last` 函数，而经典的函数式解法为递归 + 模式匹配：

```
def last[T](xs: List[T]): T = xs match {
  case Nil => throw new NoSuchElementException("last on Nil")
  case x :: Nil => x
  case _ :: xs => last(xs)
}
```

###02 Find the last but one element of a list

Example:

```
scala> penultimate(List(1, 1, 2, 3, 5, 8))
res0: Int = 5
```

```
def penultimate[T](xs: List[T]): T = xs match {
  case x :: _ :: Nil => x
  case _ :: xs => penultimate(xs)
  case _ => throw new NoSuchElementException
}
```

#### Find the lastNth element of a list
