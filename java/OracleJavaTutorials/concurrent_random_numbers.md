# 并发随机数

JDK 7 引入 `ThreadLocalRandom` 类，用于在多线程、`ForkJoinTasks` 中使用随机数。

相对 `Math.random()`，`ThreadLocalRandom` 更适合 **并发访问**，能够减少线程竞争，带来更高性能。

用法很简单，用 `ThreadLocalRandom.current()` 获取该类，然后就可以获取各种随机数：

```
int r = ThreadLocalRandom.current().nextInt(4, 77);
```
