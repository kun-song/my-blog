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

## 线程池

大部分 `Executor` 实现都使用了线程池，线程池包含 **工作者线程**，这些线程与 `Runnable` 或 `Callable` 任务 **解耦**。

使用工作者线程将 **线程创建开销** 降到最低，`Thread` 对象会使用一定内存空间，在大型应用中，大量线程会带来客观的内存管理开销。

一种常见线程池是 **固定大小** 线程池，它总是保持特定数量的线程，若线程在使用中被停止，则自动创建一个 **新线程** 来补充，任务通过 **内部队列** 提交到线程池，超出线程池容量的任务将在队列中等待。

固定大小线程池的一个重大优势是 **优雅降级**。假设有一台 HTTP 服务器，每个 HTTP 请求由一个线程处理，若为每个接收到的请求创建一个新线程，则当请求超过处理上限后，服务器会 **突然响应** 请求；若使用线程池，则不会为每个请求创建线程，线程数量是固定的，当请求过多时，并不会停止响应，而是响应 **越来越慢**。

`Executors` 提供了很多工厂方法，用于创建 `Executor`：

* `newFixedThreadPool`
  + 创建使用固定大小线程池实现的 `Executor`
* `newCachedThreadPool`
  + 容量可变线程池
  + 适用于有很多 `short-lived` 任务的场景
* `newSingleThreadExecutor`
  + 串行执行任务的 `Executor`

若上面的工厂方法不能满足需求，则可以使用 `ThreadPoolExecutor` 或 `ScheduledThreadPoolExecutor` 直接构造。

## `Fork/Join`

`fork/join` 框架是 `ExecutorService` 接口的实现，能充分利用多处理器，该框架适用于可 **递归分解** 的任务。

与其他 `ExecutorService` 实现相同，`fork/join` 框架底层采用 **线程池**，特别之处在于，该框架使用 `work-stealing` 算法。完成任务的工作者线程，可以从其他 **正在工作** 的工作者线程中 **偷取** 任务执行。

`fork/join` 框架的核心是 `ForkJoinPool` 类，该类扩展了 `AbstractExecutorService` 类，实现了 `work-stealing` 算法，且可以执行 `ForkJoinTask`。

### 基本用法

The first step for using the fork/join framework is to write code that performs a segment of the work. Your code should look similar to the following pseudocode:

使用 `fork/join` 框架的第一步是写出仅执行部分任务的代码，与下面伪代码类似：

```
if (当前任务足够小)
  直接执行任务
else
  分解任务为两份
  执行分解后的任务，等待结果
```

上面的代码通常写在 `ForkJoinTask` 的子类，比如 `RecursiveTask`（可以返回值） 或 `RecursiveAction` 中。

准备好 `ForkJoinTask` 子类后，创建一个 **代表整个任务** 的对象，将其传递给 `ForkJoinPool.invoke()` 方法。

### 例子

假设要模糊一张图片，最初的图片时一个整数数组，每个值代表一个像素的颜色值，模糊处理之后的图片用大小相同的数组表示。

Performing the blur is accomplished by working through the source array one pixel at a time. Each pixel is averaged with its surrounding pixels (the red, green, and blue components are averaged), and the result is placed in the destination array. Since an image is a large array, this process can take a long time. You can take advantage of concurrent processing on multiprocessor systems by implementing the algorithm using the fork/join framework. Here is one possible implementation:


















f
