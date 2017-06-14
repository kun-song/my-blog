# Join

`T.join` 方法会导致当前线程暂停执行，**等待线程 `T` 执行结束**。

可以为 `join` 方法指定 **等待时间**，但与 `sleep` 类似，`join` 依赖操作系统调度，所以该时间 **并不精确**。

与 `sleep` 类似，`join` 方法接收到中断时会抛出 `InterruptedException` 异常。

## Javadoc

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
