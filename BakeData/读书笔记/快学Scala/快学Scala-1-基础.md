---
title: 快学Scala-1-基础
date: 2022-01-23 02:01:26
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# Scala解释器

安装scala后，可以直接在命令行输入scala启动，之后便可以通过命令行的形式进行单条的命令执行，比如简单的数学运算。（类似python命令行用法）

![](1.png)

输出除了结果，还会显示结果类型。

技术上讲，实际发生的过程是输入内容被快速编译成字节码，之后交给JVM执行。这种在命令行即时交互的形式被称为REPL（read-eval-print-loop）

# 声明值和变量

通过val来定义常量，不可改变其值

```scala
val answer = 8*5
```
通过var来生命变量
```scala
var count = 1
count = count+1
```
scala 鼓励使用常量，除非明确知道该值回被改变
scala不需要指定类型，类型会通过初始化的表达式来推断，但必要时也可以指定：

```scala
val str1: String = "hello"
val str2: String = null
```
scala中的类型是跟在变量名称后面的，和java不同
scala中一条语句结束不用分号，只有在一行中有多条语句时才需要分号隔开

```scala
val x=10; val y=20
// 多个值或者变量一起声明
val x, y=20
```

# 常用类型

scala 有8种数值类型，Byte，Char，Short，Int，Long，Float，Double，Boolean

和java不同，这8种数值类型也是类，scala完全面向对象，不像java中还有基本类型和引用类型的区别。这表明数值类型也可以调用方法。1.toString()

除了基本的数值类型，scala还提供对应的富类型来扩展功能，有RichByte，RichChar等。

scala中类型之间的转换是由方法完成的如.toInt，而不是java中的强制类型转换。

# 算术和操作符重载

和其他语言一样，支持+-*/%和其他位运算&|^>><<

不同点是，这些操作也都是方法

a+b实际上执行的是a.+(b)

scala不支持++和--，需要用+=1和-=1

# 方法调用

scala中用 对象.方法(参数)的格式进行方法调用，如果没有参数，不要括号。scala不存在静态方法的概念，取而代之的是定义在伴生对象（object）里的方法，这些方法在引入了对应包后直接调用，不需要前缀，或者不引入，用包名来当前缀

```scala
import scala.math._
sqrt(2)

scala.math.sqrt(2)
```

# apply方法

对字符串s，s(4)表示取出第四个字符，同java中的s.charAt(4)，这种写法背后实际执行的是apply方法，即s.apply(4)。同样，BigInt("123141") 会将字符串"123141"转换为一个bigint对象，底层也是apply方法。BigInt.apply("123141")

# Scaladoc

类型javadoc，用于记录各种scala api的信息，类名旁边标注的C表示class，O表示object（伴生对象）。对于特质trait（类似java的接口），也会有t和O的标记。

