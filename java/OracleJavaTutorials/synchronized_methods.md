# 同步方法

Java 提供两种基本同步语法：同步方法、同步语句。其中同步语句稍微复杂一点，本节主要讲述同步方法。

将一个普通方法变为同步方法，只要在方法声明上添加 `synchronized` 关键字即可：

```
public class SynchronizedCounter {
    private int c = 0;

    // 同步方法
    public synchronized void increment() {
        c++;
    }

    // 同步方法
    public synchronized void decrement() {
        c--;
    }

    // 同步方法
    public synchronized int value() {
        return c;
    }
}
```

若 `count` 是 `SynchronizedCounter` 的实例对象，则上述同步方法有两个作用：

1. 多个线程调用 `count` 的同步方法，**无法交叉执行**。
  * 当线程 `A` 在 `count` 上执行 `increment` 时，因为 `increment` 是同步方法，则调用 `count` 的任意同步方法的其他线程将被挂起，直到线程 `A` 对 `increment` 的调用执行结束。
2. 同步方法执行结束后，自动与 **后续任意同步方法调用** 建立 `happens-before` 关系。
  * 保证 `count` 对象的 **状态** 对 **所有线程可见**。

> **注意** `constructor` 不能添加 `synchronized` 关键字，否则报语法错误。同步构造函数 **没有意义**，因为对象在 **构造过程中** 只能被构造它的线程访问，天然是线程安全的。

----------------------------
> **警告**
构造 **多线程共享** 的对象时，注意不要 **过早泄漏** 对象应用。假设持有一个名为 `instances` 的列表，其中保存所有类实例，若在构造函数中包括如下代码：
```
instances.add(this);
```
此时，其他线程可以在 **对象未完成构造** 时访问使用 `instances` 列表访问该对象，但此时对象并未构造完成，这时对象引用就是 **过早泄漏**。
----------------------------

  Synchronized methods enable a simple strategy for preventing thread interference and memory consistency errors: if an object is visible to more than one thread, all reads or writes to that object's variables are done through synchronized methods. (An important exception: final fields, which cannot be modified after the object is constructed, can be safely read through non-synchronized methods, once the object is constructed) This strategy is effective, but can present problems with liveness, as we'll see later in this lesson.

同步方法是一种避免 **线程干扰**、**内存一致性错误** 的简单策略：若对象对多线程可见，则对该对象 **状态** 的 **所有读、写操作** 都通过同步方法进行（重要例外：`final` 成员变量在构造之后不能再次修改，所以可以 **不通过** `synchronized` 方法方法）。

该策略有效，但会引发 **活跃性问题**。
