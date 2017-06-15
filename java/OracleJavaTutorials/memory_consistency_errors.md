# 内存一致性错误

内存一致性错误：当多个线程对 **相同数据** 持有 **不一致视图** 时发生。内存一致性错误的原因很复杂，我们不需要深究这些原因，只要知道如何有效避免这类错误就可以了。

理解内存一致性错误的关键在于理解 `happens-before` 关系，该关系是一种简单 **保证**，即一条语句执行的 **内存写** 操作对于其他语句是 **可见的**。看下面的例子，我们声明并初始化一个 `int` 值：

```
int counter = 0;
```

`counter` 被两个线程共享，假设线程 `A` 对 `counter` 做如下操作：

```
counter++;
```

之后，线程 `B` 立即打印 `counter`：

```
System.out.println(counter);
```

若上面两条语句在 **同一个线程** 中执行，我们可以安全预测打印值为 1。但若两条语句分别在 **不同线程** 中执行，则打印值可能是 0，也可能是 1，因为 **无法保证** 线程 `A` 对 `counter` 的修改操作对于线程 `B` 是可见的，除非开发者在两条语句之间建立 `happens-before` 关系。

有很多建立 `happens-before` 关系的方式，比如下面小节介绍的同步。

其实我们已经见过建立 `happens-before` 关系的几种方式：

1. 调用 `Thread.start` 语句时，所有与 `Thread.start` 语句具有 `happens-before` 关系的语句都会与 `Thread` 中的所有语句建立 `happens-before` 关系。
2. 线程 `A` 中调用 `T.join` 方法，当线程 `T` 停止，导致 `T.join` 方法返回，则线程 `T` 中的所有语句与 `T.join` 之后的语句建立 `happens-before` 关系。

总结：`Thread.start` 保证老线程的操作对新开线程可见，而 `Thread.join` 保证新开线程的操作对老线程可见。

```
public class Demo {
  public static void main(String[] args) {
    Thread t = new Thread(new SomeRunnable());

    // main 线程操作

    // 线程 t 会做很多操作
    t.start();

    t.join();

    // 处理 t 线程的结果
  }
}
```

* `Thread.start` 保证 `main` 线程在前面做的所有准备对 `t` 可见，从而可以在 `main` 线程中做些准备工作。
* `Thread.join` 保证 `t` 线程做的所有工作对 `main` 线程可见，从而可以处理 `t` 的运算结果。
