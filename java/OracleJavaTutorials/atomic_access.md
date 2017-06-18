# 原子访问

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
