---
title: 快学Scala-21-隐式转换和隐式参数
date: 2022-03-10 12:04:10
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 隐式转换

所谓隐式转换函数，指的是那种以implicit关键字声明的带有单个参数的函数。这样的函数被自动应用，将值从一种类型转换为另一种。

```scala
implicit def int2Fraction(n:Int) = Fraction(n,1)

//将调用int2Fraction(3)自动转换类型
val result = 3*Fraction(4,5)
```

# 利用隐式转换丰富现有的类库功能

在Scala中，可以定义一个警官丰富的类型，提供原本类型不包括的方法。

```scala
class RichFile(val from: File){
  def read = Source.fromFile(from.getPath).mkString
}

//再提供一个隐式转换来将原生类转换为富类
implicit def file2RichFile(from:File) = new RichFile(from)
```
这样就实现了给特定类添加方法。
# 引入隐式转换

Scala会考虑如下的隐式转换函数：

* 位于源或者目标类型的伴生对象中的隐式函数或隐式类
* 位于当前作用域中可以以单个标识符指代的隐式函数或隐式类
比如int2Fraction函数，可以放在Fraction伴生对象中。

或者，放入类中再引用这个类。

# 隐式转换规则

隐式转换在如下三种情况下会被考虑：

* 当表达式的类型与预期类型不同时
* 当对象访问一个不存在的成员时
* 当对象调用某个方法，而该方法的参数声明与传入参数不匹配时
如果代码能够在不使用隐式转换的前提下通过编译，则不会使用隐式转换。

# 隐式参数

函数或方法可以带有一个标记为implicit的参数列表，在这种情况下，编译器会查找默认值。

```scala
case class Delimiters(left:String, delims:Delimiters) = 
  delims.left + what + delims.right
  
quote("Bonjour le monde")
//此时编译器会查找一个类型为Delimiters的隐式值
```
编译器会去如下地方查找：
* 在当前作用域所有可以用单个标识符指代的满足类型要求的val和def
* 与所有要求类型相关联的类型的伴生对象和它的类型参数
# 利用隐式参数进行隐式转换

隐式的函数参数也可以被用作隐式转换

```scala
def smaller[T](a:T,b:T) = if(a<b) a else b 
// 上述用法时错误的，因为编译器不知道a和b属于一个带有<操作符的类型
// 提供一个转换函数来达到目的
def smaller[T](a:T,b:T)(implicit order: T=> Ordered[T])
= if(order(a)<b) a else b
```

# 上下文界定

类型参数可以有一个形式为T:M的上下文界定，其中M时另一个泛型类型，它要求作用域中存在一个类型为M[T]的隐式值

```scala
class Pair[T:Ordering](val first: T, val second: T){
  def smaller(implicit ord: Ordering[T]) = 
    if(ord.compare(first,second) <0) first else second
}
```
# 类型类

像上文ordering这样的特质被称为类型类，类型类定义了某种行为，任何类型都可以通过提供相应的行为来加入这个类。这里的类和面向对象编程中的类不同，可以理解为一个集体，共同包含一个功能的集体。

# 类型证明

假定编译器需要处理约束implicit ev: String <:< AnyRef，它会在伴生对象中查找类型为String<:<AnyRef的隐式对象。

我们把ev称作类型证明对象，它的存在证明了String是AnyRef的字类型。

# @implicitNotFound注解

@implicitNotFound注解告诉编译器在不能构造出带有该注解的类型的参数时给出错误提示。

# CanBuildFrom解读

CanBuildFrom特质带有一个apply方法，其交出类型为Builder[E,To]的对象。Builder类型带有一个+=方法用来将元素添加到一个内部的缓冲，还有一个result方法用来产出所要求的集合。

