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







## 忽略异常



## 捕获多个异常




## 更简单的反射方法异常
