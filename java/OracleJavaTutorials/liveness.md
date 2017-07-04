# 活跃性

并发应用能够在 **有效时间内执行** 的能力被称为活跃性（A concurrent application's `ability to execute in a timely manner` is known as its liveness.）。本章讲述三种活跃性问题：

1. 死锁
2. 饥饿
3. 活锁

## 死锁

死锁指两个或多线线程互相 **等待对方持有的资源**，从而 **永远阻塞** 的场景。

Alphonse and Gaston are friends, and great believers in courtesy. A strict rule of courtesy is that when you bow to a friend, you must remain bowed until your friend has a chance to return the bow. Unfortunately, this rule does not account for the possibility that two friends might bow to each other at the same time. This example application, Deadlock, models this possibility:

Alphonse 和 Gaston 是一对有礼貌的好朋友，礼仪中有一条严格规定：当你向朋友鞠躬时，必须保持鞠躬，直到对方回礼。不幸的是，这条规定没有考虑到两个人可能会 **同时** 向对方鞠躬，当两人同时鞠躬时，即发生死锁：

```
public class Deadlock {

    static class Friend {
        private final String name;

        public Friend(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }

        public synchronized void bow(Friend bower) {
            System.out.format("%s: %s has bowed to me!\n", this.name, bower.getName());
            // 请求 bower 的锁
            bower.bowBack(this);
        }

        public synchronized void bowBack(Friend bower) {
            System.out.format("%s: %s has bowed back to me!\n", this.name, bower.getName());
        }
    }

    public static void main(String[] args) {
        final Friend Alphonse = new Friend("Alphonse");
        final Friend Gaston = new Friend("Gaston");

        new Thread(new Runnable() {
            public void run() {
                Alphonse.bow(Gaston);
            }
        }).start();

        new Thread(new Runnable() {
            public void run() {
                Gaston.bow(Alphonse);
            }
        }).start();
    }
}
```

运行 `Deadlock`，两线程运行 `bowBack` 时，极可能互相阻塞，而且阻塞永远无法解除，因为每个线程都在等待对方停止 `bow`，以获取对方对象的内置锁。

## 饥饿与活锁

相比死锁，饥饿、活锁少见的多，但并发应用依然经常遇到。

### 饥饿

饥饿：线程 **无法获取共享资源**，无法向前推进。当 **共享资源** 被 **长期占有**，其他需要访问该资源的线程就会发生饥饿。例如若对象 `O` 中有一个耗时很长的同步方法 `m`，若线程 `T` 经常调用 `m` 方法，则其他同样需要 `m` 的线程将被阻塞。

### 活锁

线程 `A` 需要 **响应** 线程 `B` 的动作，线程 `B` 需要响应线程 `C` 的动作...此时可能会出现活锁。

* 与 **死锁** 相同：活锁中的线程 **无法向前推进**
* 与 **死锁** 不同：活锁中的线程 **没有阻塞**，只是太繁忙导致无法恢复工作
