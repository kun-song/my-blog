# Scala

## 柯里化

```
object CurryTest extends App {

  def filter(xs: List[Int], p: Int => Boolean): List[Int] =
    if (xs.isEmpty) xs
    else if (p(xs.head)) xs.head :: filter(xs.tail, p)
    else filter(xs.tail, p)

  def modN(n: Int)(x: Int) = ((x % n) == 0)

  val nums = List(1, 2, 3, 4, 5, 6, 7, 8)
  println(filter(nums, modN(2)))
  println(filter(nums, modN(3)))
}
```

`modN` 为柯里化函数，当使用部分参数调用时，返回值为：接受剩余参数作为参数的函数。

## case 类

case 类与普通类有几个关键不同点，主要用于：

1. 定义不可变数据
2. 模式匹配

### 定义

最简单的 case 类定义包括：`case class` 关键字 + 类名 + 参数列表（可以为空）：

```
case class Book(isbn: String)

val frankenstein = Book("978-0486282114")
```

* 创建 case class 对象不需要使用 `new`
* case class 的参数列表为 `val` 类型，不能重新赋值：`frankenstein.isbn = xxx` 是错误的

### 比较

case 类通过 **结构** 而非 引用进行比较：

```
case class Message(sender: String, recipient: String, body: String)

val message2 = Message("jorge@catalonia.es", "guillaume@quebec.ca", "Com va?")
val message3 = Message("jorge@catalonia.es", "guillaume@quebec.ca", "Com va?")
// true
val messagesAreTheSame = message2 == message3
```

## 模式匹配

模式匹配顾名思义，即拿 `value` 与 `pattern` 进行匹配，若匹配成功，还能对 `value` 进行 **解构**，可以替代 `switch` 和 `if-else`。

### 语法

```
import scala.util.Random

val x: Int = Random.nextInt(10)

x match {
  case 0 => "zero"
  case 1 => "one"
  case 2 => "two"
  case _ => "many"
}
```

模式匹配由三部分组成：

1. 被匹配的值 `x`
2. 关键字 `match`
3. 至少一个 `case` 表达式

### case 类 + 模式匹配

```
abstract class Notification

case class Email(sender: String, title: String, body: String) extends Notification
case class SMS(caller: String, message: String) extends Notification
case class VoiceRecording(contactName: String, link: String) extends Notification

def showNotification(notification: Notification): String = {
  notification match {
    case Email(email, title, _) =>
      s"You got an email from $email with title: $title"
    case SMS(number, message) =>
      s"You got an SMS from $number! Message: $message"
    case VoiceRecording(name, link) =>
      s"you received a Voice Recording from $name! Click the link to hear it: $link"
  }
}
val someSms = SMS("12345", "Are you there?")
val someVoiceRecording = VoiceRecording("Tom", "voicerecording.org/id/123")

println(showNotification(someSms))  // prints You got an SMS from 12345! Message: Are you there?
println(showNotification(someVoiceRecording))  // you received a Voice Recording from Tom! Click the link to hear it: voicerecording.org/id/123
```

### Pattern guards

Pattern guards 只是简单的 `if <boolean expression>`，用于 case 分支中，为模式添加约束：

```
def showImportantNotification(notification: Notification, importantPeopleInfo: Seq[String]): String = {
  notification match {
    case Email(email, _, _) if importantPeopleInfo.contains(email) =>
      "You got an email from special someone!"
    case SMS(number, _) if importantPeopleInfo.contains(number) =>
      "You got an SMS from special someone!"
    case other =>
      showNotification(other) // nothing special, delegate to our original showNotification function
  }
}

val importantPeopleInfo = Seq("867-5309", "jenny@gmail.com")

val someSms = SMS("867-5309", "Are you there?")
val someVoiceRecording = VoiceRecording("Tom", "voicerecording.org/id/123")
val importantEmail = Email("jenny@gmail.com", "Drinks tonight?", "I'm free after 5!")
val importantSms = SMS("867-5309", "I'm here! Where are you?")

println(showImportantNotification(someSms, importantPeopleInfo))
println(showImportantNotification(someVoiceRecording, importantPeopleInfo))
println(showImportantNotification(importantEmail, importantPeopleInfo))
println(showImportantNotification(importantSms, importantPeopleInfo))
```

### 只匹配类型

可以仅对 case 类的类型进行匹配：

```
abstract class Device
case class Phone(model: String) extends Device{
  def screenOff = "Turning screen off"
}
case class Computer(model: String) extends Device {
  def screenSaverOn = "Turning screen saver on..."
}

def goIdle(device: Device) = device match {
  case p: Phone => p.screenOff
  case c: Computer => c.screenSaverOn
}
```

* 当在 case 分支中需要调用对象方法时很有用
* 一般使用类型的首字母用作标识符
