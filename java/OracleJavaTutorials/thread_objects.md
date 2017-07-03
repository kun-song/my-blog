# 线程对象

每个线程都与一个 `Thread` 对象相相关，使用 `Thread` 对象创建并发系统，有两种基本策略。

1. **直接控制** 线程的创建、管理：在系统需要执行异步任务时，直接实例化 `Thread` 即可。
2. **间接管理**：将线程的管理从系统中 **抽象出来**，将异步任务传递给一个 `Executor`。

本章介绍 `Thread` 对象的使用，`Executor` 在后面讲述。

## 定义、启动线程

创建线程实例，必须提供需要在线程中执行的任务代码，有两种实现方式：

### 实现 `Runnable` 接口

`Runnable` 接口只有一个方法：`run`，该方法保存在线程中执行的代码，使用 `Runnable` 作为参数构造 `Thread` 对象：

```
public class HelloRunnable implements Runnable {
  // 任务代码
  public run() {
    System.out.println("Hello from a thread!");
  }

  public static void main(String[] args) {
    // 使用 Runnable 创建 Thread 对象
    (new Thread(new HelloRunnable())).start();
  }
}
```

注意 `Runnable` 接口无法单独运行，必须通过构造 `Thread` 对象执行。

### 继承 `Thread` 类

`Thread` 类本身实现了 `Runnable` 接口，但其 `run` 方法为空，我们可以继承 `Thread`，将 `run` 方法实现为我们的任务代码：

```
public class HelloThread extends Thread {
  // Thread 实现了 Runnable 接口，故有 run 方法
  public void run() {
    System.out.println("Hello from a thread!");
  }

  public static void main(String[] args) {
    (new HelloThread()).start();
  }
}
```

### `Runnable` vs `Thread`

上面两个例子，为了启动新线程，都调用了 `Thread.start` 方法。

实现 `Runnable` 接口更加通用，因为即使实现了 `Runnable` 接口，还可以继承其他类，而继承 `Thread` 的方式却无法再继承其他父类（Java 单继承），这减弱了其灵活性，所以推荐第一种方式。

`Thread` 类定义了其他的 **线程管理** 方法，包括获取线程信息、影响线程状态的方法。

## 暂停线程 - Sleep

The sleep method can also be used for pacing, as shown in the example that follows, and waiting for another thread with duties that are understood to have time requirements, as with the SimpleThreads example in a later section.

`Thread.sleep` 方法可以 **暂停** 当前线程，以便其他线程可以获取处理器时间，

`sleep` 方法有两个重载版本：一个以毫秒为参数，一个以纳秒为参数。有两个原因导致线程 **暂停时间无法精确保证**：

1. `sleep` 方法依赖操作系统的实现。
2. `sleep` 方法可以被中断。

永远不要依赖 `sleep` 的暂停时间。

`SleepMessages` 例子使用 `sleep` 实现每 4 秒打印一次消息：

```
public class SleepMessages {
  public static void main(String[] args) throws InterruptedException {
    String importantInfo[] = {
      "Mares eat oats",
      "Does eat oats",
      "Little lambs eat ivy",
      "A kid will eat ivy too"
    };

    for (int i = 0; i < importantInfo.length; i++) {
      // 暂停当前线程 4s
      Thread.sleep(4000);

      System.out.println(importantInfo[i]);
    }
  }
}
```

注意 `main` 方法声明抛出 `InterruptedException`，当 `sleep` **指定时间未结束**，其他线程中断该线程时，`sleep` 方法抛出 `InterruptedException`，因为例子中没有定义其他线程会引起中断，所以无需捕获该异常。

### Javadoc 注释

```
/**
  * Causes the currently executing thread to sleep (temporarily cease
  * execution) for the specified number of milliseconds, subject to
  * the precision and accuracy of system timers and schedulers. The thread
  * does not lose ownership of any monitors.
  *
  * @param  millis
  *         the length of time to sleep in milliseconds
  *
  * @throws  IllegalArgumentException
  *          if the value of {@code millis} is negative
  *
  * @throws  InterruptedException
  *          if any thread has interrupted the current thread. The
  *          <i>interrupted status</i> of the current thread is
  *          cleared when this exception is thrown.
  */
    public static native void sleep(long millis) throws InterruptedException;
```

* 受影响是 **当前线程**。
* 暂停时间受系统 **时钟**、**调度器** 的精度限制。
* **不会释放监视器锁**（这条最容易被忽视）。

## 中断

中断 **仅仅是个提示**，表明当前应该停止当前工作，做些其他事情，但线程 **如何响应中断** 完全取决于开发者，可以当做什么都没发生，但最常见的做法就是停止当前线程的执行。

要中断线程 `A`，需要在线程 `A` 上调用 `Thread.interrupt` 方法，中断机制能够正常工作，取决于被中断的线程 `A` 是否支持中断。

### 支持中断（如何响应 `InterruptedException` 异常）

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

### 中断状态标志

线程的中断机制是通过一个被称为 **中断状态** 的内部标志实现的：

1. 调用 `Thread.interrupt` 方法将会设置该标志。
2. 静态方法 `Thread.interrupted` 用于 **查询自己的中断状态**，被调用时，会清除中断状态。
3. 非静态方法 `Thread.isInterrupted` 用于 **查询其他线程的中断状态**，被调用时，不会清除被查询线程的中断状态。
4. 所有抛出 `InterruptedException` 异常的方法 `exit` 时都会清除中断状态，但该状态可能会立即被其他线程设置（通过调用 `Thread.interrupt` 方法。）。

## Join

`T.join` 方法会导致当前线程暂停执行，**等待线程 `T` 执行结束**。

可以为 `join` 方法指定 **等待时间**，但与 `sleep` 类似，`join` 依赖操作系统调度，所以该时间 **并不精确**。

与 `sleep` 类似，`join` 方法接收到中断时会抛出 `InterruptedException` 异常。

### Javadoc

```
/**
  * Waits for this thread to die.
  *
  * <p> An invocation of this method behaves in exactly the same
  * way as the invocation
  *
  * <blockquote>
  * {@linkplain #join(long) join}{@code (0)}
  * </blockquote>
  *
  * @throws  InterruptedException
  *          if any thread has interrupted the current thread. The
  *          <i>interrupted status</i> of the current thread is
  *          cleared when this exception is thrown.
  */
    public final void join() throws InterruptedException {
        join(0);
    }
```

* `t.join()` 等同于 `t.join(0)`。
* `t.join()` 等待线程 `t` 死亡。

## 例子

下面例子串联了本章介绍的概念，`SimpleThreads` 包含两个线程，第一个是主线程，主线程中使用 `Runnable` 创建了另外一个线程 `MessageLoop`，并且等待 `MessageLoop` 线程执行结束，若 `MessageLoop` 长时间不结束，主线程中断它。

`MessageLoop` 线程打印一系列消息，若在打印完所有消息之前被中断，则 `MessageLoop` 线程打印一条消息，并结束执行。

```
public class SimpleThreads {
    public static void main(String[] args) throws InterruptedException {
        // 中断 MessageLoop 线程之前等待的时间，默认 5s
        long patience = 10 * 1000;

        // 若有命令行参数，则使用，且以 s 为单位
        if (args.length > 0) {
            patience = Long.parseLong(args[0]) * 1000;
        }

        // 开始
        threadMessage("Starting MessageLoop thread");

        long startTime = System.currentTimeMillis();

        Thread t = new Thread(new MessageLoop());
        t.start();

        threadMessage("Waiting for MessageLoop thread to finish");

        while (t.isAlive()) {
            threadMessage("Still waiting...");

            // 暂停主线程的执行，1s 后检查等待时间
            t.join(1000);

            long currentTime = System.currentTimeMillis();
            // 等待时间超过 patience 则中断 MessageLoop 线程
            if ((currentTime - startTime) > patience && t.isAlive()) {
                threadMessage("Tired of waiting!");

                t.interrupt();

                t.join();
            }
        }

        // 结束
        threadMessage("Finally!");
    }

    private static class MessageLoop implements Runnable {
        public void run() {
            String[] infos = {
                    "Mares eat oats",
                    "Does eat oats",
                    "Little lambs eat ivy",
                    "A kid will eat ivy too"
            };

            for (String message : infos) {
                try {
                    Thread.sleep(4000);
                    threadMessage(message);
                } catch (InterruptedException e) {
                    // 被中断后直接返回
                    threadMessage("I wasn't done!");
                }
            }
        }
    }

    static void threadMessage(String message) {
        String threadName = Thread.currentThread().getName();
        System.out.format("%s: %s%n", threadName, message);
    }
}
```

## Thread 常用方法

### `Thread.yield` 方法

```
/**
   * A hint to the scheduler that the current thread is willing to yield
   * its current use of a processor. The scheduler is free to ignore this
   * hint.
   *
   * <p> Yield is a heuristic attempt to improve relative progression
   * between threads that would otherwise over-utilise a CPU. Its use
   * should be combined with detailed profiling and benchmarking to
   * ensure that it actually has the desired effect.
   *
   * <p> It is rarely appropriate to use this method. It may be useful
   * for debugging or testing purposes, where it may help to reproduce
   * bugs due to race conditions. It may also be useful when designing
   * concurrency control constructs such as the ones in the
   * {@link java.util.concurrent.locks} package.
   */
public static native void yield();
```
* **提示** 调度器，**当前线程** 希望放弃对处理器的使用，但调度器可以 **忽略** 该提示。
* 该方法可用于调试、测试，复现静态条件引起的 bug。
* 注意该方法为 `static` 方法，所以必须用 `Thread.yield()` 调用。

### `Thread.join` 方法

```
/**
 * Waits for this thread to die.
 *
 * <p> An invocation of this method behaves in exactly the same
 * way as the invocation
 *
 * <blockquote>
 * {@linkplain #join(long) join}{@code (0)}
 * </blockquote>
 *
 * @throws  InterruptedException
 *          if any thread has interrupted the current thread. The
 *          <i>interrupted status</i> of the current thread is
 *          cleared when this exception is thrown.
 */
public final void join() throws InterruptedException {
    join(0);
}
```
* 等待调用线程执行结束。
