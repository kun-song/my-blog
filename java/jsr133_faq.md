# JSR 133 (Java Memory Model) FAQ

原文点[这里](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)

主要内容：

1. 何为内存模型？
2. 其他语言，比如 C++，有内存模型吗？
3. JSR 133 是做什么的？
4. 重排序意味着什么？
5. 老内存模型存在什么问题？
6. 错误的同步意味着什么？
7. 同步做了什么？
8. 为何 `final` 变量有时似乎改变了自己的值？
9. `final` 变量在新 JMM 下如何工作？
10. `volatile` 是做什么的？
11. 新内存模型修复 *双重检查加锁* 的问题了吗？
12. 如果要写虚拟机该怎么做？
13. 为何要关心内存模型？

## 何为内存模型？

多处理器系统中，处理器通常有 **多级缓存**，通过加快数据访问速度（因为缓存更靠近处理器，所以速度更快）、减轻内存总线压力，缓存大大提升了处理器性能，但也带来了新的问题。比如，若两个处理器同时访问同一内存地址的内容，将发生什么？什么情况下它们会获取相同的数据？

在处理器层次，内存模型定义了必要、足够的条件，使其他处理器 **对内存的写操作** 对当前处理器 **可见**，并且当前处理器的写操作对其他处理器也是可见的。

有的处理器采用 **强内存模型**，即保证所有处理器在 **任意时刻** 对 **同一内存地址** 获取 **相同的值**。

有的采用 **弱内存模型**，即需要调用 **特殊指令**（称为 `memory barriers`）来 **flush**、**invalidate** 当前处理器的 **本地缓存**，以便当前处理器可以看到其他处理器的写、其他处理器也可以看到当前处理器的写。`memory barriers` 一般在 `lock` `unlock` 时发生，且对使用高级语言的程序员不可见。

在强内存模型下写程序要容易点，因为不需要 `memory barriers`。然而即使在 **最严格** 的内存模型下，也需要 `memory barriers`。最近的处理器设计鼓励使用若内存模型，因为通过 **放松** 对 **缓存一致性** 的要求，可以获取更大的跨处理器扩展性。

为了让 **写操作** 对其他线程可见，还需要考虑编译器对代码的 **重排序**。

比如，编译器可能认为将 **写操作** 放到程序最后会提升效率（前提是这不会改变程序语义），此时编译器 **延迟** 了写操作，其他线程只有在该操作执行后才能看到写的结果。

更可怕的是，写操作可以被 **提前**，此时其他线程在程序没有执行到写操作之前，就能看到写操作的结果。

为了获取更高性能，在 **内存模型** 限制内，允许编译器、运行时、硬件对代码 **执行顺序** 进行优化，所以会出现上面的情况。

```
class Reordering {
  int x = 0;
  int y = 0;

  public void writer() {
    // 1
    x = 1;
    // 2
    y = 2;
  }

  public void reader() {
    // 3
    int r1 = y;
    // 4
    int r2 = x;
  }
}
```

假设两个线程并发执行上述代码，且读取 `y` 时得到 2。因为对 `y` 的写在对 `x` 写之后，所以程序员可能会假设读取 `x` 时肯定获取 1。然而代码可能被重排序，所以执行顺序可能是 2 3 4 1，此时 `r1` 为 2，`r2` 为 0。

Java 内存模型描述了在多线程代码中，什么行为是合法的、进程应该怎样通过 **共享内存** 进行通信。JMM 描述了程序中的 **变量** 与从内存、寄存器中获取、存储变量的底层细节之间的关系。

Java 提供了 `volatile` `final` `synchronized` 等关键字，帮助程序员告诉编译器，该程序对 **并发** 的要求。JMM 定义了 `volatile` `synchronized` 的行为，保证正确使用 `synchronized` 的代码在所有处理器上都能正确运行。

## 其他语言，比如 C++，有内存模型吗？

多数其他语言，比如 C、C++，没有对多线程的直接支持。这些语言对重排序的限制依赖于使用的 **线程库**（比如 `pthreads`），使用的 **编译器**，以及代码运行的 **平台**。（即没有语言层面的支持，依赖外部设施）

## JSR 133 是做什么的？

自 1997 年以来，陆续发现了 JMM 的几个严重缺陷，这些缺陷或允许出现 `confusing behaviors`（比如 `final` 变量改变了自己的值），或损坏了编译器执行普通优化的能力。

JMM 目标宏大，第一次有语言试图包含一个能在不同架构上提供 **一致性并发语义** 的内存模型。但定义一致、符合直觉的内存模型比预期困难的多，JSR 133 定义了一个新的内存模型，为了修复原有模型的缺陷，`final` `volatile` 的语义 **需要修改** 。

JMM 的完整文档可以在[这里](http://www.cs.umd.edu/users/pugh/java/memoryModel)获取，即使 `synchronization` 这样简单的概念在文档中都有非常复杂的描述，幸运的是并不需要通读该文档。

JSR 133 目标如下：

* 保持已有的安全保证，比如类型安全，并加强了其他安全保证。
* 正确同步的代码语义要尽可能简单、符合直觉。
* 不完全、不正确同步的代码的潜在安全威胁要尽量小。
* 程序员可以 **推断** 出多线程程序如何与内存交互。
* 可以在广泛的硬件架构上实现高性能的 JVM。
* 初始化安全的新保证。若一个对象被正确构造（即引用没有在构造过程中逸出），则所有可见该对象引用的线程也能看到该对象中的 `final` 字段（其值在构造函数中设置），而不需要额外同步。
* 对现有代码影响最小。

## 重排序意味着什么？

有时程序变量（对象实例字段、类静态字段、数组元素）的 **实际访问顺序** 会与程序指定的顺序不同。编译器可以优化的旗号，自由改变 **指令的执行顺序**；处理器有时也会改变指令的执行顺序，数据在寄存器、处理器缓存、主内存的传递顺序可能与程序指定的顺序不同。

比如，若线程 `t` 先写入 `a` 变量，再写入 `b` 变量，且变量 `b` 不依赖变量 `a` 的值，则编译器可以自由排列这两个写入操作的顺序，而缓存可以自由的先刷新 `b` 的值，再刷新 `a` 的值。编译器、运行时、缓存都可以进行指令重排序。

编译器、运行时、硬件会创造一种 **串行假象**，即在 **单线程** 程序中，程序无法观察到重排序的影响。然而，在 **多线程** 程序中，若没有正确同步，则线程之间可以观察到对方的 **变量访问顺序** 与程序指定的不同。

大部分场景，线程 **不需要关心其他线程** 的行为，需要关心时，则使用同步。

## 老内存模型存在什么问题？

原有的内存模型有很多严重问题，它难以理解，因此常常被违反，老内存模型的晦涩、混乱促使 JSR 133 的诞生。

一个被广泛持有的误解是，若一个字段是 `final` 类型，则多线程之间 **无需同步** 就可以获取该字段的值。虽然这是非常合理的假设，但老内存模型对 `final` 字段的规定与其他普通字段 **没有任何区别** -- 即同步是保证 `final` 字段可见性的 **唯一手段**，造成的结果是线程可能会获取 `final` 字段的 **默认值**，且稍后时间段又会获取构造函数为 `final` 字段设置的值。这导致不可变对象，比如 `String`，似乎会改变自己的值，这让人很困惑。

老内存模型允许 `volatile` 写操作与非 `volatile` 读写操作重排序，这与大多数开发者的直觉相违背，从而产生困惑。

最后，可以看到程序员对于 **错误同步** 代码的执行结果的直觉往往是错误，JSR 133 希望告知程序员这个事实。

## 错误的同步意味着什么？

错误的同步代码对不同人意义不同，在 Java 内存模型中，错误的同步代码是指：

1. 线程 `t` 对变量 `x` 有写操作
2. 线程 `s` 对变量 `x` 有读操作
3. 线程中的读、写操作没有通过 **同步** 确定顺序

以上情况说明在变量 `x` 上存在数据竞争（`data race`），包含 **数据竞争** 的程序即为错误同步。

## 同步做了什么？

同步有几个方面的作用，最广为人知的是 **互斥** -- 在同一时刻，只有一个线程可以持有内置锁，以内置锁进行同步意味着，线程 `t` 进入同步代码块后，其他线程只有在 `t` 退出同步代码块时才能进入同步代码（保证对同步代码块的 **互斥访问**）。

但同步不仅仅是互斥访问，同步还能保证 **可见性**，即在同步代码之前、同步代码之中执行的 *写操作* 对于其他使用该 `monitor` 进行同步的代码都是可见的：

* 退出同步代码块后，会释放 `monitor`，导致 **缓存被刷新到主内存**，从而该线程的写操作对于其他线程都是可见的。
* 进如同步代码块钱，会获取 `monitor`，导致 `invalidate` 处理器缓存，导致变量必须从 **主内存** 读取，从而可观察其他线程的写操作。

从缓存角度看，似乎上面的情况只会影响多处理器机器，但在单处理器机器上也是一样的。

The new memory model semantics create a partial ordering on memory operations (read field, write field, lock, unlock) and other thread operations (start and join), where some actions are said to happen before other operations. When one action happens before another, the first is guaranteed to be ordered before and visible to the second. The rules of this ordering are as follows:

新内存模型在 **内存操作**（字段读、字段写、加锁、释放所）与 **线程操作**（`start` `join`）之上建立了一种 **部分有序关系**，一些操作 `happen-before` 其他操作。

当操作 `A` `happen-before` 操作 `B` 时，有如下保证：

* 操作 `A` 必然先于操作 `B` 执行
* 操作 `A` 的结果对于操作 `B` 可见

`happen-before` 规则如下：

1. Each action in a thread happens before every action in that thread that comes later in the program's order.
1. An `unlock` on a monitor happens before every subsequent lock on that same monitor.
1. A write to a `volatile` field happens before every subsequent read of that same volatile.
1. A call to `start()` on a thread happens before any actions in the started thread.
1. All actions in a thread happen before any other thread successfully returns from a `join()` on that thread.

任意内存操作，如果对 **退出同步代码之前** 的线程 `t` 可见，则在任意线程 **进入** 相同 `monitor` 保护的同步代码块后，这些操作对于 **该线程** 可见，因为所有内存操作 `happen-before` 释放锁，而释放锁 `happen-before` 获取锁。

Another implication is that the following pattern, which some people use to force a memory barrier, doesn't work:

有人用如下代码创建 `memory barrier`，但根本无效：

```
synchronized (new Object()) {}
```

上面代码其实没有做任何操作，编译器可以移除 `synchronized (new Object())`，因为其他线程无法在相同对象上进行同步（`new Object()` 创建出的对象没有引用，其他线程无法使用）。

> **注意**：多线程之间必须使用 **同一个** `monitor` 才能创建 `happens-before` 关系。

## 为何 `final` 变量有时似乎改变了自己的值？

`String` 类的实现是演示 `final` 字段值变化的最好例子。

`String` 可以用三个字段实现：字符数组、数组 `offset`、字符串长度。不同 `String` `StringBuffer` 对象之间可以 **共享字符数组**，从而减少了对象分配、赋值。比如 `String.substring()` 方法可以实现为创建一个新 `String` 对象，该对象与老对象共享字符数组，仅仅是 `length` `offset` 的值不同，对于 `String` 类，这三个字段都是 `final` 字段。

```
String s1 = "/usr/tmp";
String s2 = s1.substring(4);
```

`s2` 的 `offset` 为 4，`length` 也是 4。但在老内存模型下，其他线程获取的 `offset` 值可能是 **默认值** 0，一段时间后又获取到正确的值 4，造成的结果是 `s2` 值好像从 `/usr` 变为 `/tmp`。

老内存模型允许该行为，一些 JVM 实现禁止该行为，新内存模型认为该行为非法。

## `final` 变量在新 JMM 下如何工作？

对象的 `final` 字段的值在构造函数中设置，假设对象被正确构造（无逸出），不需要同步，在构造函数中设置的 `final` 值对 **所有线程** 可见。

In other words, do not place a reference to the object being constructed anywhere where another thread might be able to see it; do not assign it to a static field, do not register it as a listener with any other object, and so on. These tasks should be done after the constructor completes, not in the constructor.

正确构造意味着在 **构造过程中** 没有 **逸出** 本对象的引用（因此此时还未构造完成，若有引用逸出，则其他线程可以访问未构造完的对象），如下都会导致对象逸出，都应该在构造完成后做：

1. 把当前对象的引用赋值给 `static` 变量
2. 将当前对象注册为其他对象的监听器

```
class FinalFieldExample {
  final int x;
  int y;

  static FinalFieldExample f;

  public FinalFieldExample() {
    x = 3;
    y = 4;
  }

  static void writer() {
    f = new FinalFieldExample();
  }

  static void reader() {
    if (f != null) {
      int i = f.x;
      int j = f.y;
    }
  }
}
```

上面例子演示了 `final` 字段的正确使用方式，保证执行 `reader` 的线程可以获取 `f.x` 的值为 3，因为 `x` 是 `final` 字段；但不保证获取 `f.y` 的值是 4，因为 `y` 不是 `final` 字段。

若 `FinalFieldExample` 的构造函数如下：

```
public FinalFieldExample() { // bad!
  x = 3;
  y = 4;
  // 逸出
  global.obj = this;
}
```

则不保证通过 `global.obj` 获取 `x` 值的线程会获取到 3（重排序）。

能获取构造函数中设置的字段值自然很好，但若字段是 `reference` 类型，则最好也能获取引用对象（数组）的最新值。若该引用是 `final`，则也保证能获取 **引用对象** 的最新值（这里的最新值指构造函数 **结束** 后对象的值），不会出现引用时最新值，但引用的对象不是最新的情况。

不可变对象（即只包含 `final` 字段）构造完成后，若包保证该对象对 **所有线程** 都能正确可见，仍然需要使用 **同步**。

若使用 JNI 方法修改 `final` 字段的值，则行为未定义。

## `volatile` 是做什么的？




## 新内存模型修复 *双重检查加锁* 的问题了吗？




## 如果要写虚拟机该怎么做？


## 为何要关心内存模型？





























个任务
