---
title: 快学Scala-5-类
date: 2022-01-23 02:40:05
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 简单类和无参方法

简单类和Java类似

```scala
class Counter{
  private var value = 0 // 必须初始化字段
  def increment(){value+=1} // 方法默认公有
  def current() = value // 也可以 def current = value 则调用的时候也不加()
}
```
scala 中类不声明为public，所有类都公有可见，类的使用如下：
```scala
val myCounter = mew Counter
myCounter.increment()
print(myCounter.current) // 也可以myCounter.current()
```
推荐setter类型的方法加上()，getter不加

# 带getter和setter的属性

scala 对每个字段都提供get和set方法，对下面的类：

```scala
class Person{
  var age=0
}
```
scala生成的面向JVM的类，其中会有一个私有的age字段和相应的get和set方法
scala中get和set方法写作age和age_=

```scala
var fred = new Person
//getter
fred.age
//setter 
fred.age = 23
```

可以重新定义get和set方法

```scala
class Person{
  private var age = 0
  def age = age
  def age_ = (newAge : Int){
    if(newAge>age) age = newAge
  }
}
```
对Scala默认生成的getter和setter：
* 如果字段私有，则自动生成的getter和setter也是私有的，要重写公有方法才能调用
* 如果是val则没有setter
* 如果不需要生成，则将字段声明为private[this]

# 只带getter属性

用val生命的属性是只读的，没有setter。

Scala会生成一个私有的final字段和一个getter方法，没有setter

但还有一种情况，不能通过setter改变，但需要通过某种方式改变。比如自增量。

不需要setter，但是需要一个increment方法对其值进行+1操作。可以创建private的变量var，然后实现一个自增方法，而非get方法。

# 对象私有字段

scala中一个类的方法可以访问该类所有对象的私有字段，如果需要更严格的访问限制，用private[this]修饰

```scala
class Counter{
  private var value = 0;
  def isLess(other:Counter) = value < other.value 
  // 可以访问其他对象的私有属性
}
```

# Bean属性

将Scala类的字段标注为@BeanProperty时，会自动生成JavaBean标准的getXXX和setXXX方法

```scala
import scala.beans.BeanProperty

class Person{
  @BeanProperty var name: String = _
}

/** 
会默认生成四个方法：
name: String
name_=(newVal:String):Unit
getName():String
setName(newVal:String):Unit
```

![](1.png)



# 辅助构造器

和Java一样，Scala也可以有多个类构造器，但Scala中只能有一个主构造器和多个辅助构造器。

辅助构造器用法：

1. 辅助构造器的名称是this而非类名
2. 每一个辅助构造器必须以一个对先前定义的主构造器或者辅助构造器的调用开始
```scala
class Person{
  private var name = ""
  private var age = 0

  def this(name:String){
    this() // 调用主构造器
    this.name = name
  }
  
  def this(name:String,age:Int){
    this(name) // 调用上面那个构造器
    this.age = age
  }
}
```

和Java类似，没有显式定义的主构造器的类默认拥有一个无参的主构造器

```scala
val p1 = new Person
val p2 = new Person("bob")
val p3 = new Person("bob",43)
```

# 主构造器

 Scala中主构造器和类的定义交织在一起。

直接在类名后面：

```scala
class Person(val name:String, val age:Int){
  print("start")
  def description = s"new ${name}"
}
```
主构造器会执行类定义中的所有语句，当需要在构造过程中配置某个字段时，这个特性非常有用
```scala
class MyProg{
  private val props = new Properties
  props.load(new FIleReader("application.properties"))
  //构造过程中上述配置已经被加载了
}
```

如果构造参数不带val或var，则参数被升格为字段，是对象私有的不可变的，类似private[this]修饰

```scala
class Person(name:String, age:Int){
  print(name +" "+ age)
}
```


# 嵌套类

 Scala允许任何语法结构嵌套任何语法结构，比如类中定义类，函数中定义函数等

```scala
class Network{
  class Member(val name: String){
    val contacts = new ArrayBuffer[Member]
  }
  
  private val members = new ArrayBuffer[Member]
  
  def join(name: String)={
    val m = new Member(name)
    members +=m
    m
  }
}

val chatter = new Network
val myFace = new Network
```

 Scala中，每个实例都有自己的member类，chatter.Member 和 myFace.Member 是两个不同的类。

可以各自向network中添加对象，但不能跨网络添加。

```scala
val fred = chatter.join("Fred")
val wilma = chatter.join("willma")
fred.contacts += wilma //ok
val barney = myFace.join("barney")
fred.contacts += barney // 报错，barney和fred不是同一个类型
```

如果需要跨网络添加，或者让多个实例共享属性，可以用伴生对象实现（类似Java的静态）

```scala
object Network{
  class Member(val name: String){
    val contacts = new ArrayBuffer[Member]
  }
}

class Network{
  private val members = new ArrayBuffer[Network.Member]
  ....
}
```

在嵌套类中（Java中的内部类）可以用外部类.this来访问外部类的引用，进而访问外部类的属性和方法。

