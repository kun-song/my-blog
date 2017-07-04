# 原子变量

`java.util.concurrent.atomic` 包定义了支持 **原子操作** 的类，所有类都有 `set/get` 方法，其中：

* `set` 方法与 `volatile` 变量的 **写** 操作用作相同
* `get` 方法与 `volatile` 变量的 **读** 操作作用相同

即 `set` 方法与随后的 `get` 方法之间建立 `happens-before` 关系（必须是同一对象上的 `set` `get`）；`compareAndSet` 方法也是原子的，具有内存一致性的特点（`compareAndSet` 方法是原子的 **读、改、写**）。

To see how this package might be used, let's return to the Counter class we originally used to demonstrate thread interference:

前面的 `Counter` 例子代码：

```
class Counter {
  private int c = 0;

  public void increment() {
    c++;
  }

  public void decrement() {
    c--;
  }

  public int value() {
    return c;
  }
}
```

上面的 `Counter` 不是线程安全的，将其改写为线程安全对象，一种方法是使用 `synchronized` 关键字，将其所有的读、写方法都设置为同步方法：

```
class Counter {
  private int c = 0;

  public synchronized void increment() {
    c++;
  }

  public synchronized void decrement() {
    c--;
  }

  public synchronized int value() {
    return c;
  }
}
```

`Counter` 例子本身很简单，用同步可以接受，但对于 **复杂** 类，不必要的同步会导致 **活跃性** 问题，简单的将 `int` 类型替换为 `AtomicInteger` 类型，可以在不使用同步情况下，避免线程干扰（竞态条件）：

```
class AtomicInteger {
  private AtomicInteger c = new AtomicInteger(0);

  public void increment() {
    c.incrementAndGet();
  }

  public void decrement() {
    c.decrementAndGet();
  }

  public int value() {
    return c.get();
  }
}
```
