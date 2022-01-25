---
title: 快学Scala-8-继承
date: 2022-01-25 23:27:23
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 扩展类

和Java相同，用extends来扩展一个类，之后在类定义中给出子类需要，超类不具备的新属性。或者重写超类的方法。

和Java相同，声明为final的类不能被扩展，声明为final的方法不能被重写。

# 重写方法

Scala中重写方法必须使用override修饰符

```scala
class Person{
  override def toString = s"${getClass.getName}[name=$name]"
}
```
override 有助于检查出下列错误：
* 拼错了重写方法名
* 写错了重写方法的参数
* 超类中引入了新方法，但名称和子类方法抵触
和Java一样，用super关键字来调用超类方法

```scala
override def toString = s"${super.toString}[salary = $salary]"
```

# 类型检查和转换

用InstanceOf检查某个对象是否属于某个类，如果成功，则可用asInstanceOf方法将引用转换为子类的引用：

```scala
if(p.isInstanceOf[Employee]){
  val s = p.asInstanceOf[Employee] // 转换为对应类型
  ...
}
```
如果p是Employee的类或者其子类，转换会成功
如果想明确知道p是否是Employee类而非其子类，可用classOf

```scala
if(p.getClass == classOf[Employee])
```
更好的方法是用模式匹配来做
```scala
p match{
  case s:Employee => ... // 将s作为Employee处理
  case _ => // p不是EMployee
}
```

# 受保护字段和方法

和Java相同，用protected来声明方法或字段，则该方法或字段是受保护的，这样的成员可以被子类访问，但不能从其他位置看到。

和Java不同，protected的成员对类所属的包而言是不可见的，可以用包修饰符来改变。

# 超类的构造

类有一个主构造器和任意数量的辅助构造器，每个辅助构造器必须从先前定义的构造器的调用开始。

因此辅助构造器永远不能直接调用超类的构造器。

子类的辅助构造器最终会调用主构造器，只有主构造器可以调用超类的构造器。

```scala
class Employee(name:Stirng, age:Int, val salary:Double) extends
  Person(name,age)
  
// 子类Employee的主构造器调用了父类Person的构造器
```
Scala可以扩展Java类，在这种情况下，它的主构造器必须调用Java超类的某一个构造方法：
```scala
class PathWriter(p:Path, cs:Charset)extends
  java.io.PrintWriter(FIles.newBufferedWriter(p,cs))
  
```

# 重写字段

5章介绍过，Scala的字段由一个私有字段和getter/setter构成，可以用另一个同名的val字段重写一个val字段，如下面的name字段。

子类有一个私有字段和一个公有的getter方法，而这个getter方法重写了超类的getter方法：

```scala
class Person(val name: String){
  override def toStirng = s"${getClasss.getName}[name = $name]"
}
class SecretAgent(codename: String) extends Person(codename){
  override val name = "secret"
  override val toString = "secret"
}
// 重新给name赋值了
```

# 匿名子类

和Java一样，可以通过包含带有自定义或重写的代码块的方式创建一个匿名的子类

```scala
val aline = new Person("Fred"){
  def greeting = "greeting earthling, my name is Fred"
}
/**
这里val 的类型不是Person，而是person的一个匿名子类，
其中包括一个定义的greeting方法
**/
```

# 抽象类

和Java一样，可以用abstract关键字来标记不能被实例化的类，通常因为它包括没有被完整定义的方法。

```scala
abstract class Person(val name: String){
  def id: Int // 没有方法体，抽象方法，需要子类实现
}
```
在子类中对抽象方法的实现，无需使用override关键字
```scala
class Employee(name:String) extends Person(name){
  def id = name.hashCode //直接实现即可
}
```

# 抽象字段

除了抽象方法，还可以抽象字段，抽象字段就是没有初始值的字段

```scala
abstract class Person{
  val id: Int
  val name: String
}
```
具体的子类必须提供具体的字段
```scala
class Employee(val id: Int) extends Person{
  var name = ""
}
```

# 构造顺序和提前定义

考虑下面这个例子：

```scala
class Ctreture{
  val range: Int = 10
  val env: Array[Int] = enw Array[Int](range)
}

class Ant extends Creature{
  override val range = 2
}

val ant = new Ant()
```
在这个例子中，实例化一个ant对象：
* Ant的构造器在执行前先调用了超类Creature的构造器。
* 此时range值为10
* 为了初始化env，调用了range的getter
* 因为这个getter方法被重写了（Ant中 range=2）
* 此时range的getter方法交出了还未初始化的Ant类的range字段
* range方法返回0
* env的长度被设置为0
* Ant构造器执行，将range值设为2
问题出现了：最后range值为2，但env是长度为0的数组。

这里的和兴问题在于，range表达式调用了getter方法。

为了解决这个问题，需要使用提前定义语法

提前定义，就是可以让超类构造器执行之前先初始化子类的val字段

```scala
class Ant extends {override val range =2}with Creature
```

# Scala类继承关系

![](1.png)

AnyVal 和AnyRef都扩展自Any类，而Any类是根节点

Any类中定义了isInstanceOf，asInstanceOf方法，和哈希码方法。

AnyVal没有添加任何方法，只是所有值类型的一个标记

AnyRef类追加了来自Object类的监视方法wait和notify/notifyAll，还有带函数参数的synchronized

所有的Scala类都实现了ScalaObject这个标记接口，这个接口没有定义任何方法，在继承层级的另一端是nothing和null类型

null类型的唯一实例是null值，可以将null赋值给任何引用，但不能赋值给值类型的变量。比如Int不能为null。

nothing类型没有实例，这个类型对于泛型结构而言有用，空列表Nil的类型是List[Nothing]。

Nothing是所有类型的子类型。

# 对象相等性

AnyRef的eq方法检查两个引用是否指向同一个对象，AnyRef的equals方法调用eq。可以对equals进行重写。要确保重写的equals方法的类型参数是any，否则就不是重写了。

重写equals方法一般也要重新定义hashcode，被判定相等的两个对象要有一样的hashcode。

# 值类

值类具备以下特征：

1. 扩展自AnyVal
2. 主构造器有且只有一个参数，该参数是val，且没有方法体
3. 没有其他字段或构造器
4. 自动提供equals和hashcode方法
可以自己定义一个值类：

```scala
class MilTime(val time:Int) extends AnyVal{
  def minutes = time &100
  def hours = time/100
  override def toString = f"time04d"
}
```

构建new MilTime(1100)时，编译器不会分配一个新的对象，将使用背后对应的值，即1100，可以调用对应的minutes和hours方法，但不能调用Int的方法。

类比Java，值类型不会被存储在堆上，而是直接存储在方法栈上。


