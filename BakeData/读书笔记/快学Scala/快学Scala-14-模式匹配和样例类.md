---
title: 快学Scala-14-模式匹配和样例类
date: 2022-02-11 22:10:38
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 更好的switch

Scala中switch不需要添加break，default在Scala中直接用_表示

```scala
sign = ch match{
  case '+' => 1
  case '-' => -1
  case _ =>0
}

//多个条件走一个分支
prefix match{
  case "0"|"0x" |"0X" => ...  
}

//应用任意类型
color match{
  case Color.RED => ...
  case Color.BLACK => ...
}
```

# 守卫

可以在case中配置一个boolean条件，这样的case称为守卫。模式自上而下匹配，如果带守卫的这个模式没有匹配，则尝试匹配下一个模式

```scala
ch match{
  case _if Character.isDigit(ch) => digit = Character.digit(ch,10)
  case '+' => sign =1
  case '-' => sign =-1
  case '_' => sign =0
}
```

# 模式中的变量

如果case关键字后跟着一个变量名，那匹配的表达式会被赋值给这个变量

```scala
a match{
  case '+' => sign =1
  case '-' => sign = -1
  case ch => digit = Character.digit(ch,10)
}

// 注意，digit是变量，match结束后，digit会被赋值
```

# 类型模式

可以对表达式的类型进行匹配

```scala
def matchTest(x: Any): Any = x match {
  case a: Int    => a
  case s: String => Integer.parseInt(s)
  case _: BigInt => Int.MaxValue
  case _         => 0
}
```
通过模式匹配，函数的传入参数可以是不同类型，分别处理
# 匹配数组、列表和元组

在模式中使用Array表达式来匹配数组内容

```scala
arr match{
  case Array(0) => "0"
  case Array(x,y) => s"$x $y"
  case Array(0,_*) => "0 ..."
  case + => "others"
}
```

用List表达式或者::来匹配列表

```scala
lst match{
  case 0 :: Nil => "0"
  case x :: y :: Nil => s"$x $y"
  case _ => "others"
}
```

元组使用()的元组表达法

```scala
pair match{
  case (0,_) =>"0 ..."
  case (y,0) => s"$y 0"
  case _ => "others"
}
```

# 提取器

模式匹配数组，列表，元组的背后，是提取器机制——带有从对象中提取值的unapply或unapplySeq方法的对象。

```scala
arr match{
  case Array(x,0) => x
  case Array(x,rest @ _*) => rest.min
  ....
.  
}
```
Array的伴生对象就是一个提取器，定义了unapplySeq方法，该方法被调用时，以被执行匹配动作的表达式作为参数。如果调用成功，结果是一个序列的值，即数组中的值。
比如case 1中，如果数组长度为2且第二个元素为0，则将第一个值赋给x。

提取器还被用在正则表达式中，如果正则表达式分组，可以用提取器来匹配每个分组

```scala
val pattern = "([0-9]+)([a-z]+)".r
"99 bottles" match{
  case pattern(num,item) => ...
}
// 99 和 bottles 被赋给了 num 和item
```

# 变量声明中的模式

可以在变量声明中使用模式

```scala
val (x,y) = (1,2)
val (q,r) = BigInt(10) /% 3
val Array(first,second, rest @ _*) = arr
```
这里其实是一种简写，省略了case中的部分
# for表达式中的模式

可以在for表达式中使用带变量的模式，对每一个遍历到的值，这些变量会被绑定，可以方便的遍历映射

```scala
for((k,v) <- System.getProperties())
  print(s"$k $v")
 
// 添加守卫
for((k,v) <- System.getProperties() if v=="")
  print(k)
  
```

# 样例类

样例类经过优化用于模式匹配，在class前加case关键字

```scala
abstract class Amount
case class Dollar(value:Double) extends Amount
case class Currrency(value:Double, unit:String) extends Amount

// 单例样例类
case object Nothing extends Amount
```

单例类可以使用在模式匹配中，判断当前对象到底属于哪个子类，并且可以对属性值进行操作

```scala
amt match{
  case Dollar(v) => s"$$$v"
  case Currency(_,u) => ...
  case Nothing => ""
}
```

样例类的特别之处在于如下事件自动发生：

1. 构造器中的每个参数都称为val
2. 伴生对象中提供apply方法，不用new关键字就能构造出相应的对象
3. 提供unapply方法让给你模式匹配可以工作
4. 生成toString，equals，hashCode和copy方法
# copy方法和带名参数

样例类的copy方法创建一个与现有对象值相同的新对象

```scala
val amt = Currency(22,"EUR")
val price = amt.copy()
```

可以用带名参数来修改属性

```scala
val price = amt.copy(value = 10)
```

# case语句中的中置表示法

如果unapply的返回结果是一个对偶，可以在case语句中使用中置表示法。

```scala
amt match {case a Currency u => ...}
//等价 case Currency(a,u)

lst match {case h::t => ...}
//等价 case ::(h,t)
```

# 匹配嵌套结构

样例类常被用于嵌套结构，即样例类的某个属性是另一个样例类

```scala
abstract class Item
case class Article(desc:String, price:Double) extends Item
case class Bundle(desc:String, discount:Double, itmes: Item*) extends Item
```

定义嵌套对象不用new，很容易

```scala
Bundle("a",12,Article("b",14),Article("c",14),Bundle()...)
```

模式可以匹配到特定的嵌套

```scala
case Bouldle(_,_,Article(descr,_),_*) =>
```
# 样例类是邪恶的吗

样例类适用于那种标记不会改变的结构，比如Scala中的list，列表要么是空的，要么一头一尾，不会增加一个新的样例。或者对应的Bean，也适合用样例类实现。

# 密封类

使用样例类来做模式匹配时，可能想让编译器检查确保已经列出来所有可能的选择。要达到这个目的，需要将样例类的通用超类声明为sealed

```scala
sealed abstract class Amount
case class Dollar(value:Double) extends Amount
case class Currrency(value:Double, unit:String) extends Amount
```

密封类的所有子类必须在与该密封类相同的文件中定义。比如要新加一个欧元类，则必须加在声明Amout类的文件里。

如果某个类是密封的，那么在编译期所有子类就是可知的。编译器因此可以检查模式匹配的完整性。

# 模拟枚举

通过样例类在Scala中模拟出枚举类型

```scala
sealed abstract class Light
case object Red extends Light
case object Yellow extends Light
case object Greeen extends Light

color match{
  case Red => ..
  case Yellow => ...
  case Green => ....
}
```
枚举类还可以用Enumeration类来实现
# Option类型

标准类库中的Option类型用样例类来表示可能存在也可能不存在的值。样例子类Some包装了某个值，比如Some("fff")，样例对象None表示没有值。

Option支持范型，上述some的类型为Option[String]。

map类的get方法返回一个Option，如果给定的键没有值，则返回None。如果有则值包在Some里返回。

getOrElse方法就是一个模式匹配的简单写法，包括了Some和None两种情况下的处理。

# 偏函数

花括号内的一组case语句是一个偏函数（partial function），一个并非对所有输入值都有定义的函数。

偏函数是PartialFuction[A,B]类的一个实例，A是参数，B是返回类型。

该类有两个方法，apply方法从匹配到模式计算函数的值。isDefinedAt方法在输入至少匹配其中一个模式时返回true。

