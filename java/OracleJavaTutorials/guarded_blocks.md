# Guarded Blocks

线程之间经常需要协调他们的行为，最常用的协调语法是 `guarded block`，保护块在进一步执行前，会 **轮询** 必须满足的条件。

假设 `guardedJoy` 方法只有在 `joy` 为 `false` 时才能向前执行，而 `joy` 是共享变量，由其他线程设置。`guardedJoy` 方法可以不断循环，直到条件满足，但这样会浪费 CPU 资源。

```
public void guardedJoy() {
    // 不断循环，浪费 CPU 资源
    while(!joy) {}
    System.out.println("Joy has been achieved!");
}
```

一个更加高效的方式是使用 `Object.wait` 方法暂停当前线程，`wait` 方法调用后，**不会立刻返回**，而是 **暂停** 当前线程，直到另外一个线程发出事件通知（不一定是 `guardedJoy` 期待的事件）。

```
public synchronized void guardedJoy() {
    // 每接收到一个事件通知，才循环判断一次，直到 joy 满足条件
    while(!joy) {
        try {
            wait();
        } catch (InterruptedException e) {}
    }
    System.out.println("Joy and efficiency have been achieved!");
}
```

> **注意**：一定要在循环体内使用 `wait`，因为使 `wait` 恢复执行的事件通知并不一定是程序期待的，所以每次通知到来时都要检测下 `joy` 的值。

与其他暂停线程的方法相同，`wait` 方法会抛出 `InterruptedException` 异常，本例中，因为只关心 `joy` 的值，所以忽略了该异常。

为什么上面这版 `guardedJoy` 方法是同步的？假设 `d` 是调用 `wait` 方法的对象，则当一个线程调用 `d.wait()` 时，该线程 **必须拥有 `d` 的内置锁**，否则会发生错误，调用 `wait` 方法本身这个动作将会获取对象的内置锁。

`wait` 调用之后，该线程 **释放锁**，并 **暂停执行**。在将来某个时间，另外一个线程将获取相同的锁，并调用 `Object.notifyAll` 方法，该方法将会通知 **所有等待该锁** 的线程。

```
public synchronized notifyJoy() {
    joy = true;
    notifyAll();
}
```

等到第二个线程 **释放锁** 后，第一个线程将 **重新获取锁**，并从 `wait` 方法结束处 **恢复执行**。

> **注意**：`Object` 有另外一个通知方法，`notify`，该方法仅唤醒一个线程。因为 `notify` 方法 **无法指定被唤醒的线程**，所以仅在大型并行应用中有用（有大量线程执行相似任务，并不关心具体哪个线程被唤醒）。

可以用保护块创建“生产者-消费者应用”，其中有两个线程 `producer` 和 `consumer`，他们共享数据，线程之间通过共享对象通信，两个线程之间的协同条件如下：

1. `consumer` 不能在 `producer` 生产数据之前消费。
2. `producer` 不能在 `consumer` 消费结束之前生产。

本例中，`consumer` 和 `producer` 共享的数据是一系列文本消息，用 `Drop` 对象表示：

```
public class Drop {
  // producer 发送给 consumer 的消息
  private String message;

  // 1. true 表示 consumer 需要等待 producer 发送数据
  // 2. false 表示 producer 需要等待 consumer 消费数据
  private boolean empty = true;

  public synchronized String take() {
    // 等待消息到来
    while(empty) {
      try {
        wait();
      } catch (InterruptedException e) {}
    }

    // 修改状态
    empty = true;

    // 通知 producer 状态已改变，可以发送消息了
    notifyAll();

    return message;
  }

  public synchronized void put(String message) {
    // 等待消息为空
    while(! empty) {
      try {
        wait();
      } catch(InterruptedException e) {}
    }

    // 修改状态
    empty = false;

    // 保存消息
    this.message = message;

    // 通知 consumer 状态已改变，可以消费数据了
    notifyAll();
  }
}
```

`Producer` 线程会发送一系列文本消息，字符串 `DONE` 表示所有消息发送完毕。为了还原真实世界的不可预测性，`Producer` 线程在发送消息之间暂停随机时间：

```
public class Producer implements Runnable {
  // 线程共享数据
  private Drop drop;

  public Producer(Drop drop) {
    this.drop = drop;
  }

  public void run() {
    String importantInfo[] = {
      "Mares eat oats",
      "Does eat oats",
      "Little lambs eat ivy",
      "A kid will eat ivy too"
    };

    Random random = new Random();

    for (String message : importantInfo) {
      // 发送消息
      drop.put(message);

      try {
        Thread.sleep(random.nextInt(5000));
      } catch(InterruptedException e) {}
    }
    // 消息结束标志
    drop.put("DONE");
  }
}
```

`Consumer` 线程获取消息并打印，直到遇到 "DONE" 字符串：

```
class Consumer implements Runnable {
  // 线程共享数据
  private Drop drop;

  public Consumer(Drop drop) {
    this.drop = drop;
  }

  public void run() {
    Random random = new Random();

    for (String message = drop.take(); ! message.equals("DONE"); message = drop.take()) {
      System.out.formate("MESSAGE RECEIVED: %s%n", message);

      try {
        Thread.sleep(random.nextInt(5000));
      } catch(InterruptedException e) {}
    }
  }
}
```

最后是主线程，将启动 `Producer` 和 `Consumer` 线程：

```
public class ProducerConsumerDemo {
  public static void main(String[] args) {
    // 线程共享变量
    Drop drop = new Drop();

    (new Thread(new Producer(drop))).start();
    (new Thread(new Consumer(drop))).start();
  }
}

```

> **注意**：`Drop` 类仅供演示保护块的使用，在实际应用中，不要重复造轮子，使用集合框架中的数据结构，而不是自己发明。
