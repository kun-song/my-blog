# 同步 Synchronization


Java 线程间通信主要通过 **资源共享**，资源包括 `fields` 和 `reference fields` 引用的对象。这种通信方式非常高效，但容易导致两类错误：

1. 线程干扰
2. 内存一致性错误

Java 通过 **同步** 来避免上面的错误。

然而，同步会引入 **线程竞争**，即两个以上线程同时访问同一资源，导致 Java 线程执行 **变慢**，甚至 **暂停线程执行**。线程竞争有两种表现形式：

1. 饥饿
2. 活锁

本章包括以下主题：

1. **线程干扰** 介绍多个线程访问共享数据时，错误是如何引入的。
2. **内存一致性错误** 介绍由 **共享内存** 的 **不一致视图** 引入的错误。
3. **同步方法** 介绍避免线程干扰、内存一致性错误的简单语法：`synchronized` 关键字。
4. **隐式锁与同步** 介绍更加通用的同步语法，描述如何通过 **隐式锁** 实现同步。
5. **原子访问** 介绍不能被其他线程干扰的操作。

## 线程干扰（竞态条件）

考虑如下类：

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

`Counter` 类被设计为调用 `increment` 将对 `c` 加 1，调用 `decrement` 将对 `c` 减 1。但若 `Counter` 对象被多个线程同时访问，线程之间的干扰将造成意想不到的结果。

线程干扰发生场景：

1. 多个线程共享数据
2. 每个线程对数据做不同操作（操作由多个步骤组成，一般是修改、写入操作，读操作不会出现干扰）
3. 不同操作的步骤 **交叉执行**

看起来对 `Counter` 对象的操作不可能交叉执行，因为每个对 `c` 的操作都只是一个简单语句。但在 Java 虚拟机中，**一条语句** 会转换为 **多个步骤**，比如 `c++` 这个语句将被 JVM 转换为如下三个步骤：

1. 获取 `c` 的当前值
2. 对 `c` 的当前值加一
3. 将加 1 后的值保存到 `c` 中

简化为三步：`read` -> `operation` -> `store`。

假设线程 `A` `B` 分别同时调用 `increment` 和 `decrement`，则可能出现如下的执行序列：

Thread A: Retrieve c.
Thread B: Retrieve c.
Thread A: Increment retrieved value; result is 1.
Thread B: Decrement retrieved value; result is -1.
Thread A: Store result in c; c is now 1.
Thread B: Store result in c; c is now -1.

线程 `A` 的结果被 `B` 的结果覆盖，从而丢失，上面的序列只是一种可能，在其他可能的场景下，可能 `B` 的结果丢失，也可能没有错误。因为结果是无法预测的，所以线程干扰造成的错误很难排查。

### 竞态条件

线程干扰这种说法不常见，一般称竞态条件，即：多个线程竞争对共享资源的访问（修改、写入），程序执行结果取决于各线程 **操作时序**，因而无法确定结果。

**解决办法**

同步。

## 内存一致性错误

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

## 同步方法

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

## 内置锁与同步

Synchronization is built around an internal entity known as the intrinsic lock or monitor lock. (The API specification often refers to this entity simply as a "monitor.") Intrinsic locks play a role in both aspects of synchronization: enforcing exclusive access to an object's state and establishing happens-before relationships that are essential to visibility.

同步是建立在 `intrinsic lock`/`monitor lock` 基础上的。内置锁在同步的两个方面都有作用：

1. 对于对象 **状态** 的 **排他性访问**。
2. 建立 `happens-before` 关系（内存可见性）。

**每个对象** 都有一个内置锁。若线程需要实现对对象的 **排他性** 和 **一致性** 访问，则必须在访问之前 **获取内置锁**，在访问结束后 **释放内置锁**。在获取之后、释放之前，成为 **持有** 内置锁。**只要有一个** 线程持有对象的内置锁，其他线程都无法获相同的锁，其他线程若尝试获取，则会导致 **线程被阻塞**。

线程释放内置锁后，该动作与后续所有获取 **相同锁** 的动作建立 `happens-before` 关系。

### 同步方法中的锁

线程调用同步方法时，会自动获取该方法 **所在对象** 的内置锁，在方法执行结束后，释放该锁。即使该方法是由于 **抛出异常** 而结束，也会 **释放内置锁**。

`static synchronized method` 有一点特殊，因为静态方法是 **属于类** 的，所以访问 `static synchronized method` 的线程获取的内置锁，是 **Class 对象的内置锁**，该锁不同于 **对象实例的内置锁**。

### 同步语句

与同步方法不同，同步语句必须 **显式** 指定 **提供内置锁的对象**：

```
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }

    nameList.add(name);
}
```

上述例子中，`addName` 方法中对 `lastName` 和 `nameCount` 的操作必须是同步的，同时 **尽量避免** 对其他对象的方法进行同步（比如，`nameList.add(name)` 方法，因为从同步方法中调用 **其他对象** 的同步方法可能会导致 **活跃性** 问题），若无同步语句，则只能把 `nameList.add(name)` 抽离成独立的非同步方法。

Synchronized statements are also useful for improving concurrency with fine-grained synchronization. Suppose, for example, class MsLunch has two instance fields, c1 and c2, that are never used together. All updates of these fields must be synchronized, but there's no reason to prevent an update of c1 from being interleaved with an update of c2 — and doing so reduces concurrency by creating unnecessary blocking. Instead of using synchronized methods or otherwise using the lock associated with this, we create two objects solely to provide locks.

同步语句相比同步方法，具备 **细粒度** 的同步，可以提高并发性。假设 `MsLunch` 有两个成员变量 `c1` `c2`，它俩从来不会同时使用。对成员变量的所有更新都必须同步，但没有理由阻止对 `c1` `c2` 的更新交叉执行，而且这会引入不必须的阻塞，降低并发性。通过显示提供两个锁可以解决该问题：

```
public class MsLunch {
    private long c1 = 0;
    private long c2 = 0;

    private Object lock1 = new Object();
    private Object lock2 = new Object();

    public void inc1() {
        synchronized(lock1) {
            c1++;
        }
    }

    public void inc2() {
        synchronized(lock2) {
            c2++;
        }
    }
}
```
* 此时对 `c1` `c2` 的操作可以交叉执行了，但首先一定要确保这样做是安全的。

### 可重入同步

线程无法获取其他线程持有的锁，但可以获取 **自己已经持有** 的锁。允许线程多次获取相同锁引出 **reentrant synchronization**，可重入同步指，一段 **同步代码**（同步方法 + 同步语句），直接或者间接，调用了其他 **同步代码**，但这两段同步代码使用了 **同一个锁**，若无可重入同步，同步代码需要很多冗余代码，来避免因为线程获取已持有的锁，而导致的阻塞。

## 原子访问

原子操作：

* 要么完全成功，要么完全失败，没有中间状态。
* 执行过程中 **无副作用**，直到执行完成后才能看到副作用。

We have already seen that an increment expression, such as c++, does not describe an atomic action. Even very simple expressions can define complex actions that can decompose into other actions. However, there are actions you can specify that are atomic:

即使简单的 `c++` 也可以分解为多个步骤，并非原子操作。虽然大部分操作都不是原子操作，但下面两个必然是原子的：

1. 读、写 `reference variables` 和大部分 `primitive variables`（除 `long` `double` 以外）。
2. 读、写 `volatile variables`（包括 `long` `double`）。

原子操作（实际指组成原子操作的多个步骤）无法交叉执行，因此原子操作不会出现 **线程干扰**，但还会出现 **内存一致性错误**，所以原子操作有时也需要 **同步**。使用 `volatile` 变量 **降低** 了发生内存一致性错误的风险，因为对于 `volatile` 变量的 **任何修改** 对 **所有线程可见**。另外，当一个线程读取 `volatile` 变量时，线程不仅能获取 `volatile` 变量的 **最新值**，而且可以看到导致 `volatile` 变量变化的代码的副作用。

* 同步代码块可以避免 **线程干扰** 和 **内存一致性错误**。
* 原子变量只能避免 **线程干扰**，还需要额外分析以避免出现内存一致性错误。

相比同步代码块，原子变量访问更加高效，要根据应用的规模、复杂性选择。
