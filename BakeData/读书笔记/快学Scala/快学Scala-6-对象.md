---
title: 快学Scala-6-对象
date: 2022-01-23 02:40:24
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 单例对象

 Scala没有静态方法或静态字段，可以用object这个语法结构来达到同样的目的，对象定义了某个类的单个实例，和Java中静态满足的特性相同

```scala
object Accounts{
  private var lastNumber = 0
  def newUniqueNumber() = {
    lastNumber +=1
    lastNumber
  }
}
```

调用Accounts.newUniqueNumber()就可以每次获得一个递增的id，只有当这个方法首次被调用的时候，这个对象才会被构造。

Scala中的对象可以完成任何Java中需要使用单例对象的地方

* 存放工具函数或者常量
* 共享单个不可变实例
* 单个实例来协调某个服务（单例模式）
# 伴生对象

Java中有的类会同时包括了静态方法和实例方法，Scala中可以用伴生对象的方式实现静态方法。与类同名的对象定义称为伴生对象

```scala
class Account{
  val id = Account.newUniqueNumber()
  private var balance = 0.0
  def deposit(amount: Double) {balance += amount}
  ...
}

object Account { // 伴生对象
  private var lastNumber = 0
  private def newUniqueNumber() = {
    lastNumber +=1
    lasterNumber
  }
} 
```
类和其伴生对象可以相互访问私有特性，它们必须存在同一个源文件中。
注意，类的伴生对象并不在类的作用域里，因此方法不能直接调用， 要加前缀，比如Account类中调用newUniqueNumber方法则需要写为Account.newUniqueNumber()

# 扩展类或特质的对象

一个object可以扩展类（类似继承）以及一个或多个特质（类似接口），这样获得的对象拥有扩展功能，同时又拥有类中定义的功能。

```scala
//定义一个可撤销动作类
abstract class UndoableAction(val description: String){
  def undo():Unit
  def redo():Unit
}
// 定义一个实例来扩展撤销动作
object DoNothingAction extends UndoableAction("Do nothing"){
  override def undo(){}
  override def redo(){}
}
```

# apply方法

当遇到Object(param1,param2..)的方法，就是调用了对象的apply方法，会返回一个伴生类的对象。

```scala
// 这样调用构造函数this，返回一个类实例
new Array(1000)

// 这样调用apply方法，返回伴生对象
Array(1000)

//可以在伴生对象中重写apply方法
object Account{
  def apply(id:Int){
    new Account(id)
  }
}
```

# 应用程序对象

每个Scala程序都必须从一个main方法开始

```scala
object Hello{
  def main(args:Array[String]){
    print("hello world")
  }
}

// 扩展app特征，将程序代码放入构造器方法体内：
object Hello extends App{
  print("hello world")
}

//通过args得到命令行参数
object Hello extends App{
  if(args.length>0)
    print(f"hello ${args(0)}")
  else
    print("hello word")
}
```

通过扩展App类，来隐藏了main方法

# 枚举

Scala没有枚举类型，通过Enumeration助手类产出枚举。

```scala
object Color extends Enumertaion{
  val red,yellow,green = value
}

//初始化
val red = Value(1,"stop")
val yellow = Value(2,"wait")
val green = Value(3,"go")

//调用
Color.red // 拿到一个类，再通过.来获取相应的属性值
```



