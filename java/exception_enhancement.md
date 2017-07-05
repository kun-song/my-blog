# Java 7 异常改进

Java 7 对异常处理的增强主要有以下几点：

1. `try-with-resource`
2. 捕获多重异常

## `try-with-resource`

Java 7 以前，若需要 **关闭资源**，通常需要繁琐的操作：

```
private static void printFiles(String filePath) {
    FileInputStream in = null;
    try {
        // 1. 抛出 FileNotFoundException
        in = new FileInputStream(filePath);

        // 2. 抛出 IOException
        int data = in.read();
        while (data != -1) {
            System.out.print((char) data);
            // 2. 抛出 IOException
            data = in.read();
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (in != null) {
            try {
                // 3. 抛出 IOException
                in.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## `try-with-resource`

### 古老的 `try-catch-finally`

Java 7 以前，必须 **显式关闭资源**，语法很繁琐，比如一个读取文件内容并打印的程序：

```
private static void printFile(String filePath) throws IOException {
  // 1.
  FileInputStream in = null;

  try {
    in = new FileInputStream(filePath);
    // 2.
    int data = in.read();
    while (data != -1) {
      System.out.println((char) data);
      // 3.
      data = in.read();
    }
  } finally {
    if (in != null) {
      // 4.
      in.close();
    }
  }
}
```

上述例子有 4 处地方会抛出异常：`try` 有 3 个，`finally` 有 1 个。无论 `try` 是否抛出异常，都会执行 `finally`，在 `finally` 中会试图关闭 `in`，当关闭失败时，也会抛出异常。

**问题** 当 `try` 中抛出异常，且 `finally` 关闭资源也抛出异常时，到底哪个异常会传播到调用者呢？

* `try` 中的异常是调用者关心的，因为这说明了到底发生了什么。`finally` 中的异常，并非 **真正出问题** 的地方。
* 但在 `try-catch-finally` 中，`finally` 中的异常会向上传播，这并不是我们想要的。

### 改进的 `try-with-resource`

在 Java 7 中，提供了更好的 **资源关闭** 方案：`try-with-resource`：

```
private static void printFileBetter(String filePath) throws IOException {
  try (FileInputStream in = new FileInputStream(filePath)) {
    int data = in.read();
    while (data != -1) {
      Sytem.out.print((char) data);
      data = in.read();
    }
  }
}
```

* `try` 执行结束后，里面的 `in` 资源 **自动关闭**。
* `try` 中抛出异常 `S`，关闭 `in` 资源时抛出异常 `T`，则异常 `S` 会向上传播，`T` 被抑制，这与传统的 `try-catch-finally` 区别很大。
* 任何实现了 `java.lang.AutoCloseable` 接口的资源都可以用于 `try-with-resource`。

### 在 `try-with-resource` 中使用多个资源

`try-with-resource` 中可以使用多个资源，这些资源都会 **自动关闭**，且关闭顺序是与其 **创建顺序** 相反。

```
private static void printFile(String filePath) throws IOException {
  try (FileInputStream in = new FileInputStream(filePath);
      BufferedInputStream br = new BufferedInputStream(in)) {
        int data = br.read();
        while (data != -1) {
          Sytem.out.print((char) data);
          data = br.read();
        }
      }
}
```
* 先关闭 `br`，再关闭 `in`

### 使用 `catch` `finally`

`try-with-resource` 也可以包含 `catch` `finally` 分支，两者都会在 **关闭资源** 之后执行。

在单个 `try-with-resource` 中不要放置太多逻辑代码。

## 捕获多重异常

在 Java 7 中，可以在一个 `catch` 分支中捕获多个异常，这便于我们将 **处理逻辑相同** 的异常，放在一个 `catch` 中处理。
