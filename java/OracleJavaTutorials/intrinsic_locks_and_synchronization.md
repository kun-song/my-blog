# 内置锁与同步

Synchronization is built around an internal entity known as the intrinsic lock or monitor lock. (The API specification often refers to this entity simply as a "monitor.") Intrinsic locks play a role in both aspects of synchronization: enforcing exclusive access to an object's state and establishing happens-before relationships that are essential to visibility.

同步是建立在 `intrinsic lock`/`monitor lock` 基础上的。内置锁在同步的两个方面都有作用：

1. 对于对象 **状态** 的 **排他性访问**。
2. 建立 `happens-before` 关系（内存可见性）。

**每个对象** 都有一个内置锁。若线程需要实现对对象的 **排他性** 和 **一致性** 访问，则必须在访问之前 **获取内置锁**，在访问结束后 **释放内置锁**。在获取之后、释放之前，成为 **持有** 内置锁。**只要有一个** 线程持有对象的内置锁，其他线程都无法获相同的锁，其他线程若尝试获取，则会导致 **线程被阻塞**。

线程释放内置锁后，该动作与后续所有获取 **相同锁** 的动作建立 `happens-before` 关系。

## 同步方法中的锁

线程调用同步方法时，会自动获取该方法 **所在对象** 的内置锁，在方法执行结束后，释放该锁。即使该方法是由于 **抛出异常** 而结束，也会 **释放内置锁**。

`static synchronized method` 有一点特殊，因为静态方法是 **属于类** 的，所以访问 `static synchronized method` 的线程获取的内置锁，是 **Class 对象的内置锁**，该锁不同于 **对象实例的内置锁**。

## 同步语句

与同步方法不同，同步语句必须 **显式** 指定 **提供内置锁的对象**：

```
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }

    nameList.add(name);
}
```

上述例子中，`addName` 方法中对 `lastName` 和 `nameCount` 的操作必须是同步的，同时 **尽量避免** 对其他对象的方法进行同步（比如，`nameList.add(name)` 方法，因为从同步方法中调用 **其他对象** 的同步方法可能会导致 **活跃性** 问题），若无同步语句，则只能把 `nameList.add(name)` 抽离成独立的非同步方法。

Synchronized statements are also useful for improving concurrency with fine-grained synchronization. Suppose, for example, class MsLunch has two instance fields, c1 and c2, that are never used together. All updates of these fields must be synchronized, but there's no reason to prevent an update of c1 from being interleaved with an update of c2 — and doing so reduces concurrency by creating unnecessary blocking. Instead of using synchronized methods or otherwise using the lock associated with this, we create two objects solely to provide locks.

同步语句相比同步方法，具备 **细粒度** 的同步，可以提高并发性。假设 `MsLunch` 有两个成员变量 `c1` `c2`，它俩从来不会同时使用。对成员变量的所有更新都必须同步，但没有理由阻止对 `c1` `c2` 的更新交叉执行，而且这会引入不必须的阻塞，降低并发性。通过显示提供两个锁可以解决该问题：

```
public class MsLunch {
    private long c1 = 0;
    private long c2 = 0;

    private Object lock1 = new Object();
    private Object lock2 = new Object();

    public void inc1() {
        synchronized(lock1) {
            c1++;
        }
    }

    public void inc2() {
        synchronized(lock2) {
            c2++;
        }
    }
}
```
* 此时对 `c1` `c2` 的操作可以交叉执行了，但首先一定要确保这样做是安全的。

## 可重入同步

线程无法获取其他线程持有的锁，但可以获取 **自己已经持有** 的锁。允许线程多次获取相同锁引出 **reentrant synchronization**，可重入同步指，一段 **同步代码**（同步方法 + 同步语句），直接或者间接，调用了其他 **同步代码**，但这两段同步代码使用了 **同一个锁**，若无可重入同步，同步代码需要很多冗余代码，来避免因为线程获取已持有的锁，而导致的阻塞。
