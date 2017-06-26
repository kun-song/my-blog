# Executor

迄今为止所有例子，都存在两个概念：
1. 任务（定义在 `Runnable` 接口中）
2. 线程（`Thread`）

且两者联系紧密（使用 `Runnable` 作为接口创建 `Thread`）。应用逐渐复杂后，有必要将 **线程创建、管理** 从应用中抽象出来，Java 中 `Executor` 负责线程管理。

* `Executor` 接口定义了三种 `Executor` 对象。
* `Executor` 一般使用 **线程池** 实现。
* `Fork/Join` 框架可以很好的利用多核处理器。
