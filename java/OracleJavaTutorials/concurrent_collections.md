# 并发集合

Java 并法包包含对 Java 集合框架的补充，分类如下：

* `BlockingQueue`
  + 先进先出队列
  + 可阻塞、超时
* `ConcurrentMap`
  + 继承 `Map` 接口，添加 **原子操作**，从而避免使用 **同步**
  + `ConcurrentHashMap` 实现该接口，是 `HashMap` 的并发版本
* `ConcurrentNavigableMap`
  + 继承 `ConcurrentMap` 接口，且支持 **近似匹配**
  + `ConcurrentSkipListMap` 实现该接口，是 `TreeMap` 的并发版本

上述集合类在 **添加** 操作、随后的 **读**、**删除** 操作之间建立 `happens-before` 关系，可避免 **内存一致性错误**。
