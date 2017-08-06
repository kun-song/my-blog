# SBT 指南

本课程评分系统使用 sbt 进行构建、测试、运行，本指南内容包括：

1. SBT 基本用法
2. 如何使用 SBT 提交作业任务

## 基本用法

### Base or project's root directory

基本目录（根目录），指包含指定项目的目录，即 `build.sbt` 文件所在目录。

### Source code

源代码可以直接放在根目录中，但这很杂乱，不推荐。

SBT 使用与 Maven 相同的目录结构：

```
src/
  main/
    resources/
       <files to include in main jar here>
    scala/
       <main Scala sources>
    java/
       <main Java sources>
  test/
    resources
       <files to include in test jar here>
    scala/
       <test Scala sources>
    java/
       <test Java sources>
Other directories in src/ will be ignored. Additionally, all hidden
directories will be ignored.
```

### SBT build definition files

You’ve already seen `build.sbt` in the project’s base directory. Other sbt files appear in a project subdirectory.

The project folder can contain .scala files, which are combined with .sbt files to form the complete build definition. See organizing the build for more.

## SBT 任务

### 启动 SBT

通过终端进入 `build.sbt` 文件所在目录，执行 sbt：

```
# This is the shell of the operating system
$ cd /path/to/parprog-project-directory
$ sbt
# This is the sbt shell
>
```

### 在 SBT 终端中运行 Scala 解释器

在 sbt 终端中可以打开 `Scala` 解释器（REPL），当从 sbt 中运行 REPL 时，该项目中的 **所有代码** 都会被加载，进而在 REPL 中可以访问，这也是只有在无编译错误时才能（从 sbt）进入 REPL 的原因。

使用 `Ctrl+D` 退出解释器。

### 编译

编译 `src/main/scala` 目录中的源文件：

```
> compile
[info] Compiling 4 Scala sources to /Users/aleksandar/example/target/scala-2.11
  /classes...
[success] Total time: 1 s, completed Mar 21, 2013 11:04:46 PM
>
```

### 单元测试

`test` 任务可以运行 `src/test/scala` 目录中的单元测试：

```
> test
[info] ListsSuite:
[info] - one plus one is two
[info] - sum of a few numbers *** FAILED ***
[info]   3 did not equal 2 (ListsSuite.scala:23)
[info] - max of a few numbers
[error] Failed: : Total 3, Failed 1, Errors 0, Passed 2, Skipped 0
[error] Failed tests:
[error]   example.ListsSuite
[error] {file:/Users/luc/example/}assignment/test:test: Tests unsuccessful
[error] Total time: 5 s, completed Aug 10, 2012 10:19:53 PM
>
```

### 运行

若项目包含 `main` 方法、或者有对象继承 `App` 特性，则 `run` 可以运行它们，当有多个 `main` 方法时，会提示你选择一个运行：

```
> run
Multiple main classes detected, select one to run:

 [1] example.Lists
 [2] example.M2

Enter number: 1

[info] Running example.Lists
main method!
[success] Total time: 33 s, completed Aug 10, 2012 10:25:06 PM
>
```

## 提交代码到 Cousera

sbt 可以将源码打包为 jar 文件，并上传到 Cousera 的服务器（只有无编译错误的代码才能提交）。

The submit tasks takes two arguments: your Coursera `e-mail address` and the `submission token`.

```
> submit e-mail@university.org suBmISsioNPasSwoRd
[info] Packaging /Users/luc/example/target/scala-2.11/parprog-example_2.11-1.0.0-sources.jar ...
[info] Done packaging.
[info] Compiling 1 Scala source to /Users/luc/example/target/scala-2.11/classes...
[info] Connecting to coursera. Obtaining challenge...
[info] Computing challenge response...
[info] Submitting solution...
[success] Your code was successfully submitted: Your submission has been accepted and will be graded shortly.
[success] Total time: 6 s, completed Aug 10, 2012 10:35:53 PM
>
```

也可以在未启动 sbt 终端时提交：

```
$ sbt "submit e-mail@university.org suBmISsioNPasSwoRd"
```
