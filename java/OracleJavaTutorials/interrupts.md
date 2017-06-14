# 中断

中断 **仅仅是个提示**，表明当前应该停止当前工作，做些其他事情，但线程 **如何响应中断** 完全取决于开发者，可以当做什么都没发生，但最常见的做法就是停止当前线程的执行。

要中断线程 `A`，需要在线程 `A` 上调用 `Thread.interrupt` 方法，中断机制能够正常工作，取决于被中断的线程 `A` 是否支持中断。

## 支持中断（如何响应 `InterruptedException` 异常）

线程如何支持中断完全取决于线程如何设计，若线程 `A` 频繁调用抛出 `InterruptedException` 异常的方法，则可以在捕获到中断异常时 `return` 即可。假设 `SleepMessages` 中的 `for` 循环在一个线程的 `run` 方法中，则可以如下修改以支持影响中断。

```
for (int i = 0; i < importantInfo.length; i++) {
    // 暂停当前线程 4s
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        // 支持中断：被中断时停止打印消息
        return;
    }
    // 打印消息
    System.out.println(importantInfo[i]);
}
```

许多抛出 `InterruptedException` 异常的方法，比如 `sleep`，被设计为接收到中断时，取消当前操作，立即返回。

若线程没有调用可以抛出 `InterruptedException` 异常的方法，但该线程又会执行很长时间怎么办？此时，该线程必须周期性调用 `Thread.interrupted` 方法，该方法在接收到中断时为 `true`：

```
for (int i = 0; i < inputs.length; i++) {
    // 极度耗时操作
    heavyCrunch(inputs[i]);

    // 周期性检查当前线程是否被中断
    if (Thread.interrupted()) {
        // 被中断时停止执行，立刻返回
        return;
    }
}
```

* 因为上面线程中没有调用能抛出 `InterruptedException` 异常的方法，但其他线程可以调用该线程的 `interrupt` 方法从而中断上面的线程，所以必须周期检查该线程是否被中断。
* 并非调用抛出 `InterruptedException` 方法的线程才会被中断，**所有线程都可被中断**。

上面的例子很简单，检测到中断后直接返回，在复杂应用中，将中断异常再次抛出更加合理：

```
if (Thread.interrupted()) {
    throw new InterruptedException();
}
```
* 重新抛出异常可以让异常处理代码 **集中在一个 try-catch 中**。
* `Thread.interrupted` 会清除当前线程的中断状态。

## 中断状态标志

线程的中断机制是通过一个被称为 **中断状态** 的内部标志实现的：

1. 调用 `Thread.interrupt` 方法将会设置该标志。
2. 静态方法 `Thread.interrupted` 用于 **查询自己的中断状态**，被调用时，会清除中断状态。
3. 非静态方法 `Thread.isInterrupted` 用于 **查询其他线程的中断状态**，被调用时，不会清除被查询线程的中断状态。
4. 所有抛出 `InterruptedException` 异常的方法 `exit` 时都会清除中断状态，但该状态可能会立即被其他线程设置（通过调用 `Thread.interrupt` 方法。）。
