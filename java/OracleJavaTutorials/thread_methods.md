# Thread 常用方法

## `Thread.yield` 方法

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

## `Thread.join` 方法

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
