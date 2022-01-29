---
title: 快学Scala-10-特质
date: 2022-01-29 23:36:28
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 为什么没有多重继承

和Java相同，scala也不允许多重继承，如果允许，则当这些父类有了共同的字段或者方法时，子类对应字段或方法的获取就会存在歧义。

比如助教类继承了老师和学生两个类，并且老师和学生两个类都有自己的id，那么助教的id到底应该取哪个？

还有一种情况，假设老师和学生两个类都继承自人这个类，那就产生了菱形继承问题。

![](1.png)

这种情况下如何合并name字段，又如何构造呢？

在Java中，类只能扩展自一个父类，可以实现任意数量的接口，但接口只能包括抽象方法，静态方法和默认方法，不能包括字段（属性）。

Java的默认方法有局限性，它可以调用其他接口的方法，但不能使用对象状态。因此，Java中经常要同时提供接口和抽象基类。如果要同时扩展两个抽象基类就不行了。

Scala提供特质trait而非接口，特质可以同时拥有抽象方法和具体方法，以及状态。

而类可以实现多个特质，这个设计解决了Java的接口问题。

# 当作接口使用的特质

特质的用法和Java接口类似

```scala
trait Logger{
  def log(msg:String)
}
```
类可以实现特质，并且特质中的方法不需要标注为abstract，未实现的方法默认就是抽象的
```scala
class ConsoleLogger extends Logger{ //extends 而不是implement
  def log(msg:String){
    print(msg)
  } // override 也不用写
}
```
用with添加多个trait
```scala
class ConsoleLogger extends Logger with Cloneable with Serializable
```
所有Java接口都可以当成Scala的特质来实现
# 带有具体实现的特质

特质中的方法不一定是抽象的，特质中可以包括带有实现的方法，而扩展了这个特质的类可以直接调用这些实现的方法。

```scala
trait ConsoleLogger{
  def log(msg:String) {print(msg)}
}

class SavingsAccount extends Account with ConsoleLogger{
  def withdraw(amount: Double){
    if(amount>balance) log("Insufficient funds")
    else balance -= amount
  }
}
```

# 带有特质的对象

先构造一个抽象类添加特质

```scala
abstract class SavingsAccount extends Account with Logger{
  def withdraw(amount: Double){
    if(amount>balance) log("Insufficient funds")
    else balance -= amount
  }
}
```
抽象类无法直接实例化，可以在构造对象的时候混入一个具体的方法实现，通过这种方式实现抽象方法。
```scala
trait ConsoleLogger extends Logger{
  def log(msg:String) {print(msg)}
}

val acct = new SavingsAccount with ConsoleLogger

//直接在对象层面加入不同特质，实现不同方法
val acct2 = new SavingsAccount with FileLogger
```

# 叠加在一起的特质

可以为类或者对象添加多个互相调用的特质，从最后一个开始，适用于分阶段加工处理某个值的场景。

当我们想给所有日志添加时间戳时：

```scala
trait TimestampLogger extends ConsoleLogger{
  override def log(msg:String){
    super.log(s"${java.time.Instant.now()} $msg")
  }
}
```
想修改过长的日志：
```scala
trait shortLog extends ConsoleLogger{
  override def log(msg:String){
    if(msg.length>10)msg = msg.subString(0,10)
    super.log(s"$msg")
  }
}
```
一下两种对象的初始化，在打印日志时输出结果完全不同
```scala
val acc1 = new SavingsAccount with TimestampLogger with ShortLogger
val acc2 = new SavingsAccount with ShortLog with TimestampLogger
```

acc1就会先截取，后加时间戳

acc2就会先加时间戳，后截取。

在简单的混入序列中，特质是从后往前生效的，即后面的特质先处理。

# 在特质中重写抽象方法

上面两个特质都调用了扩展特质ConsoleLogger的log方法。但如果这里的log方法是抽象的，比如直接扩展了Logger，则报错。

如果想直接调用这种抽象方法，则新的特质中方法也是抽象的，必须加上abstract和override关键字

```scala
abstract override def log(msg:String){
//此处的log方法是logger里抽象方法
  super.log(s"$msg....")
}
```

# 当作富接口使用的特质

特质可以包含大量工具方法，这些方法可以依赖一些抽象方法来实现。

```scala
trait Logger{
  def log(msg:String)
  def info(msg:String) {log(s"Info: $msg")}
  def warn(msg:String) {log(s"Warn: $msg")}
  def servere(msg: String){log(s"SEVERE: $msg")}
}
```
之后扩展了Logger特质的类就可以调用这些方法了。注意，如果仍不实现log方法，只能在抽象类中调用，否则要实现这个抽象方法。
```scala
class Test extends Logger{
  def log(msg:String) = {print(msg)}
  def getWarn(msg:String) = warn(msg)
  def getInfo(msg:String) = info(msg)
}
```

# 特质中的具体字段

特质中可以包含字段，这个字段也可以是具体的或者抽象的。如果给出了初始值，则字段就是具体的。

```scala
trait ShortLogger extends Logger{
  val maxLength = 15
  abstract override def log(msg:String){
    super.log(
    if(msg.length<=maxLength) msg
    else s"${msg.substring(0,maxLength-3)}..."
    )
  }
}
```
注意，特质中的字段只是被加到了使用该特质的类中，而不是继承关系。具体区别在于继承自超类的字段属于超类对象，当超类改变时，子类不需要重新编译就会拿到最新的值（引用关系），而特质内的字段会被归为该类自己的字段，当特质改变时，所有扩展了这个特质的类都需要重新编译来改变这个字段的值。
# 特质中的抽象字段

特质中未被初始化的字段在具体的子类中必须被重写。

```scala
class Test extends ShortLogger{
 val maxLength = 20 // 重写字段
}
```
# 特质构造顺序

特质也有构造器，由字段初始化和特质体中其他语句构成

```scala
trait FileLogger extends Logger{
  val out = new PrintWriter("app.log")
  out.println(s"a new logger")
  //上面两句就是构造体的一部分，在构造特质的时候会执行
  def log(msg:String){...}
}
```
语句在任何混入该特质的对象构造时会被执行
执行顺序如下：

1. 调用超类的构造器
2. 特质构造器在超类构造器后，类构造器前执行
3. 特质由左到右构造
4. 父特质先构造（多个特质有相同的夫特质，只构造一次）
5. 子类最后构造
```scala
class SavingAccount extends Account wiht FileLogger with ShortLogger
```
1.  Acount (超类)
2. Logger（父特质）
3. FileLogger（左一特质）
4. ShortLogger（左二特质）
5. SavingAccount（类）
# 初始化特质中的字段

特质不能有构造器参数，每个特质都带有一个默认的无参构造器。

缺少构造器参数时类和特质唯一的区别

如果需要初始化时给特质中的字段赋值，好的做法是将特质中需要使用某个字段的值定义为懒值lazy，之后在初始化后对特质中的值进行赋值。

```scala
trait FileLogger extends Logger{
  val fileName:String
  lazy val out = new PrintStream(filename)
  def log(msg:String){...}
}
```

# 扩展类的特质

特质也可以扩展类，这个类将自动成为所有混入了该特质的超类

```scala
trait LoggedException extends Exception with ConsoleLogger{
  def log(){log(getMessage())}
}

class TestException extends LoggedException{
  override def getMessage() ="aaa!"
}
```
特质扩展了超类Exception，并且调用了其方法getMessage()，TestException类在混入这个特质的时候自动成为类Exception的子类。
# 自身类型

混入特质时，编译器要确保所有混入该特质的类都能把这个类当作超类。Scala还可以通过自身类型的机制保证这一点。当特质以如下代码开始定义时，就只能被混入指定类型的子类中。

```scala
trait LoggedExcepption extends ConsoleLogger{
  this:Exception =>
    def log(){log(getMessage())}
}
```
这个特质并不扩展Exception类，而是有一个自身类型Exception，这意味着只能被混入Exception的子类。
# 背后发生了什么

只有抽象方法的特质被简单的变成一个Java接口

特质方法对应的是Java的默认方法

如果有特质字段，对应的Java接口就有getter和settter方法

