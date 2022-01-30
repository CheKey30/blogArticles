---
title: 快学Scala-11-操作符
date: 2022-01-30 23:02:51
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 标识符

变量，函数，类等到名称统称为标识符，命名规则推荐和Java保持一致。对于保留关键字，也可以当作标识符来使用，但需要反引号来区分

```scala
val `val` = 4
Thread.`yield()`
```

# 中置操作符

可以将函数标识符放在两个量中间使用，称为中置操作符。

```scala
1 to 10
1.to(10)

1 ->10
1.->(10)
```
中置操作符有两个参数，前后都是。
# 一元操作符

只有一个参数的操作符称为一元操作符，比如+，-，!，～可以放在参数之前做前置操作符，这些操作符只有一个参数，具体调用了unary_操作符的方法。

还有后置操作符，也是一元的。

```scala
-a
a.unary_-

a toString
a.toString()
```

# 赋值操作符

操作符= 被称为赋值操作符

```scala
a+=b
a = a+b
```
注意，<=,>=,!=不是赋值操作符
===，==，=/=也不是赋值操作符

# 优先级

在没有括号的情况下，高优先级的操作符先执行。Scala可以随意定义操作符，因此有一套自己的优先级规则。除了赋值操作符之外，优先级由操作符的首字符决定：

![](1.png)

* 同一行字符产出的操作符优先级相同。如+和->优先级相同
* 后置低于中置
# 结合性

当有一系列相同优先级的操作符时，操作符的结合性决定了是从左往右还是从右往左求值。

Scala中操作符都是左结合的（从左往右），比如19-77+2.除了：

* 冒号结尾的操作符
* 赋值操作符
比如

```scala
1::2::Nil
//创建出包含2的列表，这个列表作为尾部拼接到1作为头部的列表中
```

# apply和update方法

apply和upate方法用于对映射和数组的取值或赋值

```scala
val scores = new scala.collection.mutable.HashMap[String,Int]
scores("Bob") = 100 // scores.update("Bob",100)
val bobScore = scores("Bob") // scores.apply("Bob")
```

# 提取器

提取器是一个带有unapply方法的对象，可以把unapply当作伴生对象中apply的反向操作。

apply接受构造参数，将他们变成对象。而unapply方法接受一个对象，然后从中提取值，当初用来构造对象的值。

```scala
var Fraction(a,b) = Fraction(3,4)*Fraction(5,6)
// 将两个fraction对象的值解析出来，重新构建一个fraction对象

case Fraction(a,b) => ... //模式匹配
```
可以在对象中定义unapply方法，来提取任何构造参数
```scala
object Name{
  def unapply(input: String)={
    val pos = input.indexof(" ")
    if(pos ==-1) None
    else Some((input.substring(0,pos),input.substring(pos+1)))
  }
}
```

# 带单个参数或无参数的提取器

 Scala中没有只带一个组件的元组，如果用unapply方法要提取单值，则它应该返回一个目标类型的Option

```scala
object Number{
  def unapply(input:String): Option[Int] =
  try{
    Some(input.trim.toInt)
  }  catch{
    case ex: NumberFormatException => None
  }
}

//提取字符串成数字
val Nomber(n) = "1729"
```
也可定义提取器返回boolean
```scala
object IsCompound {
  def unapply(input: String) = input.contains(" ")
}
```

# unapplySeq方法

提取任意长度的值的序列，应该用unapplySeq来命名方法返回一个Option[Seq[A]]，A是被提取值的类型。

```scala
object Name{
  def unapplySeq(input:String): Option[Seq[String]] = 
    if(input.trim == "") None else Some(input.trim.split("\\s+"))
}

//匹配任意数量的变量
author match{
  case Name(first,last) => "aa bb"
  case Name(first,middle,last) => "aa bb cc"
}
```

# 动态调用

 Scala是强类型的语言，在编译期而不是运行期会报告类型错误。如果x.f(args)过了编译，则说明x一定有一个方法f。

Scala中，如果某个类型扩展自scala.Dynamic 这个特质，则它的方法调用，getter和setter都会被重写成对特殊方法对调用，这些特殊方法可以检视原始调用的方法名称和参数，并采取任意行动。相当于实现了在程序运行期定义方法。

```scala
class DynamicProps(val props: java.util.Properties) extends Dynamic{
  ...
  def applyDynamicNamed(name: String)(args:(String,String)*){
    if(name !="add") throw new IllegalArgumentException
      for((k,v)<-args)
        props.setProperty(k.replaceAll("_","."),v)
  }
}
```

