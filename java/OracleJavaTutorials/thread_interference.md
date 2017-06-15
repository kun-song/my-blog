# 线程干扰


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
2. 每个线程对数据做不同操作（操作由多个步骤组成）
3. 不同操作的步骤 **交叉执行**

It might not seem possible for operations on instances of Counter to interleave, since both operations on c are single, simple statements. However, even simple statements can translate to multiple steps by the virtual machine. We won't examine the specific steps the virtual machine takes — it is enough to know that the single expression c++ can be decomposed into three steps:

看起来对 `Counter` 对象的操作不可能交叉执行，因为每个对 `c` 的操作都只是一个简单语句。但在 Java 虚拟机中，**一条语句** 会转换为 **多个步骤**，比如 `c++` 这个语句将被 JVM 转换为如下三个步骤：

1. 获取 `c` 的当前值
2. 对 `c` 的当前值加一
3. 将加 1 后的值保存到 `c` 中

简化为三步：`read` -> `operation` -> `store`。

Suppose Thread A invokes increment at about the same time Thread B invokes decrement. If the initial value of c is 0, their interleaved actions might follow this sequence:

假设线程 `A` `B` 分别同时调用 `increment` 和 `decrement`，则可能出现如下的执行序列：

Thread A: Retrieve c.
Thread B: Retrieve c.
Thread A: Increment retrieved value; result is 1.
Thread B: Decrement retrieved value; result is -1.
Thread A: Store result in c; c is now 1.
Thread B: Store result in c; c is now -1.

线程 `A` 的结果被 `B` 的结果覆盖，从而丢失，上面的序列只是一种可能，在其他可能的场景下，可能 `B` 的结果丢失，也可能没有错误。因为结果是无法预测的，所以线程干扰造成的错误很难排查。
