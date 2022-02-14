---
title: 快学Scala-15-注解
date: 2022-02-14 23:47:46
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 什么是注解

注解是加入到代码中以便有工具可以对其进行处理的标签。工具可以在代码级别运作，也可以处理被编译器加入了注解信息的类文件。

@Test，@Override 都是注解。

Scala支持Java的注解，也有Scala独有的注解。注解会影响编译过程，比如@BeanProperty会触发getter和setter

# 什么可以被注解

Scala可以为类，方法，字段，局部变量和参数添加注解，和Java一样。

```scala
@Entity class Credentials
@Test def testSomeFeature(){}
@BeaProperty var username = _
def doSome(@NotNull message : String){}

// 多个注解先后顺序没有影响
@BeanProperty @Id var username = _

// 主构造器的注解如果没有参数需要加括号
class Credentials @Inject()(var username: String, var password: String)

// 为表达式添加注解，要加在表达式:后面
(myMap.get(key): @unchecked) match{...}

// 为类型参数添加注解
class MyContainer[@specialized T]

// 针对实际类型的注解放在类型后
def country: String @Localized
```

# 注解参数

注解可以有带名参数，如果只有一个value参数可省略名字。不带参数则取默认值。

Scala的注解参数类型可以是任意的

```scala
@Test(timeout = 100, expected = classOf[IOException])
@Named("creds") var credentials: Credentials  =_
```

# 注解实现

可以自己通过注解类实现自己的注解，但不常用。

注解必须扩展Annotation 特质，类型注解必须扩展自TypeAnnotation特质

```scala
class unchecked extends annotation.Annotation

class Localized extends StaticAnnotation with TypeConstrain
```

# 针对Java特性的注解

Scala提供了一组用于和Java互操作的注解

## Java修饰符

不常用的Java特性，Scala用注解

```scala
@volatile var done = false
@transient var re = new HashMap[String,String]
```

## 标记接口

Scala用@Cloneable和@remote来标记可以被克隆和远程的对象

用@SerialVersionUID来标记序列化和指定序列化版本

## 受检异常

Java编译器会跟踪受检异常，如果从Java中调用Scala方法，需要用@throws来生成正确的签名。

Java编译器需要知道方法可抛出的异常种类，否则会拒绝捕获该异常。

## 变长参数

@varargs注解可以从Java调用Scala带有变长参数的方法。

```scala
@varags def process(args: String*)
//可以被Java调用，编译器生成如下方法
void process(String... args) //Java桥接方法
```

## JavaBeans

@scala.reflect.BeanProperty注解使得编译器生成JavaBeans风格的getter和setter方法

```scala
class Person{
  @BeanProperty var name : String = _
}
```

# 用于优化的注解

 Scala中的部分注解用于控制编译器优化

## 尾递归

@tailrec注解让编译器尝试将递归调用的函数转换为循环，提高效率。如果不能转换，则会报错。

```scala
class Util{
  @tailrec def sum2(xs:Seq[Int], partial:BigInt): BigInt =
    if(xs.isEmpty) partial else sum2(xs.head + partial)
}
```

## 跳转表生成与内联

Java中，switch语句会被编译成跳转表，比一系列if高效。@switch注解在Scala中用于检查Scala的match语句是不是被编译成了跳转表。

```scala
(n:@switch) match{
  case 0 => ..
  case 1 => ..
  case 2 => ..
}
```

内联指将方法调用语句替换为被调用的方法体。可以用@inline在建议编译器做内联。或@noinline来禁止内联。

## 可省略方法

@elidable给可以在生产代码中移除的方法打标记。类似于日志输出级别

## 基本类型的特殊化

可以让编译器自动生成范型方法的重载方法，省去打包解包基本类型的损耗。

```scala
def allDiff[T](x:T,y:T,z:T) = x!=y && x!=z && y!=z

//编译器自动生成不同基本类型的范型方法
def allDiff[@specialized T](x:T,y:T,z:T) = x!=y && x!=z && y!=z
```

# 用于错误和警告的注解

@deprecated注解会让编译器遇到时生成一个警告信息

```scala
@deprecated(message="use factorial")
def factorila(n:Int):Int = ...
```
可被用在提醒函数以过时等
@Unchecked用于跳过匹配完整检查

