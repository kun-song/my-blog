# 定义、启动线程

创建线程实例，必须提供需要在线程中执行的任务代码，有两种实现方式：

## 实现 `Runnable` 接口

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

## 继承 `Thread` 类

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

## `Runnable` vs `Thread`

上面两个例子，为了启动新线程，都调用了 `Thread.start` 方法。

实现 `Runnable` 接口更加通用，因为即使实现了 `Runnable` 接口，还可以继承其他类，而继承 `Thread` 的方式却无法再继承其他父类（Java 单继承），这减弱了其灵活性，所以推荐第一种方式。

`Thread` 类定义了其他的 **线程管理** 方法，包括获取线程信息、影响线程状态的方法。
