# Executor

迄今为止所有例子，都存在两个概念：
1. 任务（定义在 `Runnable` 接口中）
2. 线程（`Thread`）

且两者联系紧密（使用 `Runnable` 作为接口创建 `Thread`）。应用逐渐复杂后，有必要将 **线程创建、管理** 从应用中抽象出来，Java 中 `Executor` 负责线程管理。

* `Executor` 接口定义了三种 `Executor` 对象。
* `Executor` 一般使用 **线程池** 实现。
* `Fork/Join` 框架可以很好的利用多核处理器。

## Executor 接口

有三个 `Executor` 接口：

1. `Executor`
  * 最简单、支持发起任务
1. `ExecutorService`
  * 继承 `Executor` 接口，添加任务、`Executor` **生命周期** 的管理方法
1. `ScheduledExecutorService`
  * 继承 `ExecutorService` 接口，支持 `future`、按周期支持任务

通常将 `Executor` 对象声明为以上三种接口之一，而不直接声明为 `Executor` 实现类。

### `Executor`

`Executor` 接口只有一个 `execute` 方法，用来 **取代** 线程创建语法。若 `r` 为 `Runnable` 对象，`e` 为 `Executor` 对象，则可用：

```
e.execute(r);
```

取代：

```
(new Thread(r)).start();
```

直接使用 `Thread` 会创建线程，且立刻执行线程；但 `Executor.execute()` 方法则依赖于具体实现：

* 可能与直接使用 `Thread` 相同，也会立刻创建线程、执行线程；
* 可能使用 **线程池** 中已有的工作者线程执行任务（省去线程创建）；
* 可能进入队列，等待工作者线程可用；

并发包中的 `Executor` 实现一般都是实现的 `ExecutorService` 或 `ScheduledExecutorService` 接口，但用 `Executor` 接口也能使用。

### `ExecutorService`

`ExecutorService` 接口添加了功能更加全面的 `submit` 方法（与 `execute` 功能类似）：

* `submit` 可以接受 `Runnable` 或 `Callable` 对象（可以有返回值）
* `submit` 返回 `Future` 对象，用于
  + 获取 `Callable` 的返回值
  + 管理 `Callable`/`Runnable` 任务的状态

`ExecutorService` 支持 **批量添加** `Callable` 任务；也提供了关闭 `Executor` 的方法，若要支持立即关闭，任务需要正确处理中断。

### `ScheduledExecutorService`

`ScheduledExecutorService` 在 `ExecutorService` 之上添加：

* `schedule` 方法：支持在 **指定时延** 后执行 `Runnable`/`Callable` 任务；
* `scheduleAtFixedRate`/`scheduleWithFixedDelay` 方法：支持以 **指定间隔** 周期执行任务；
