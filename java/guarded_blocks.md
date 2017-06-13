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

一个更加高效的方式是使用 `Object.wait` 方法暂停当前线程，`wait` 方法调用后，**不会立刻返回**，而是 **暂停** 当前线程，直到另外一个线程发出事件通知（不一定是 `guardedJoy` 期待的通知）。

```
public synchronized void guardedJoy() {
    // 每接收到一个事件通知，就循环判断一次，直到 joy 满足条件
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
