# Join

`T.join` 方法会导致当前线程暂停执行，**等待线程 `T` 执行结束**。

可以为 `join` 方法指定 **等待时间**，但与 `sleep` 类似，`join` 依赖操作系统调度，所以该时间 **并不精确**。

与 `sleep` 类似，`join` 方法接收到中断时会抛出 `InterruptedException` 异常。
