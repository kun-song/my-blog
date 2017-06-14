# 暂停线程 - Sleep

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

## Javadoc 注释

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
