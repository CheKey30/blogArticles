---
title: 快学Scala-12-高阶函数
date: 2022-02-09 23:14:39
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 作为值的函数

 Scala中可以在变量中存储函数

```scala
import scala.math._
val num = 3.14
val fun = ceil _
```
将num设置为3.14，将fun设置为一个函数_表明确实指定的是函数，而不是少传了参数。num的类型为Double，而fun的类型是(Double) => Double; 接受并返回Double的函数。
可以对函数进行调用或者传递

```scala
//调用
fun(num)
//传递
Array(3.1,3.2,3.3).map(fun)
//map方法接受一个函数参数，将它应用到数组中的每一个值上面
```

# 匿名函数

Scala中无需对每一个函数命名

```scala
//匿名函数
(x:Double) => 3*x

//将匿名函数存在变量里
val triple = (x:Doulbe) => 3*x

//等效于用def来定义的方法
def triple(x:Double) = 3*x

//可以直接将匿名函数进行传递
.....map((x:Double) => 3*x)

//也可以用花括号
....map{(x:Double) => 3*x}
```
def 定义的是方法，而不是函数
# 带函数参数的函数

函数/方法可以接受另一个函数做参数

```scala
def valueAtOneQuarter(f:(Double) => Double) = f(0.25)
//此处参数可以是任何以Doble为输入和输出的函数
```
这个方法的类型是((Double)=>Double) => Double
这类接受函数做参数的方法/函数就是高阶函数

高阶函数也可以产出另一个函数

```scala
def mulBy(factor:Double) = (x:Double) =>factor*x

mulBy(3) //返回函数(x: Double) => 3*x

val quintuple = mulBy(5)
// quintuple 就是一个乘5的函数
```

# 参数（类型）推断

将匿名函数传递给另一个函数方法时，Scala会帮助推断类型信息，上面方法可以简写

```scala
valueAtOneQuarter((x)=>3*x)
//内部的匿名函数可以省略类型，因为方法已经定义了参数类型

//单个参数还能省略括号
valueAtOneQuarter(x=>3*x)

//右侧函数体内如果参数只出现一次，可以用_代替
valueAtOneQuarter(3*_)
```

# 一些有用的高阶函数

* map方法就是一个高阶函数，会对集合中的所有元素应用传入的方法
* foreach方法和map相似，但是foreach中的函数没有返回值，只讲函数应用到每一个值比如打印
* filter方法对集合元素进行筛选，传入一个boolean类型的函数
* reduceLeft方法接受一个二元函数（带有两个参数的函数），并将它应用在序列中的所有元素上
```scala
(1 to 6).reduceLeft(_*_) // 返回1*2*3*4*5*6
```
* sortWith接受一个boolean类型的二元函数，比较大小，对序列进行排序
# 闭包

Scala中可以在任何作用域内定义函数，包，类甚至另一个函数或方法里都可以。

函数体内可以访问到相应作用域的任何变量。并且函数可以在变量不再处于作用域内的时候调用。

```scala
def mulBy(factor:Double) = (x:Double) =>factor*x
val triple = mulBy(3)
val half = mulBy(0.5)
println(s"${triple(14)} ${half(14)}") // 打印42 7
```
执行过程如下：
1. mulBy首次调用将参数factor设置为3，之后被函数引用，存入triple，之后参数变量factor从运行时的栈上弹出
2. multy再次被调用，factor设置为0.5，被函数引用，存为half。factor弹出
每一个返回的函数都有自己的factor设置

这样的一个函数被称为闭包（closure），闭包由代码和代码用到的任何非局部变量定义构成。

这些函数实际以类的对象形式实现，该类由一个实例变量factor和一个包含了函数体的apply方法构成。

# SAM转换

从Scala 2.12开始，可以将Scala函数传给预期接受一个“SAM接口”（任何带有单个抽象方法的Java接口）的Java代码。Java中这类接口称作函数式接口。

```scala
button.addctionListener(event => counter+=1)
```

# 柯里化

指将原来接受两个参数的函数转化为新的接受一个参数的函数的过程。新的函数返回一个以原有第二个参数作为参数的函数。

```scala
//两个参数的函数
val mul = (x:Int, y:Int) => x*y
//柯里化，接受一个参数，生成另一个接受单个参数的函数
val mulOneAtTime =(x:Int) =>((y:Int) => x*y)
// 如下调用
mulOneAtTime(6)(7)
//mulOneAtTime(6)的结果是一个函数(y:Int) => 6*y
```
Scala支持多组括号的形式来实现柯里化
```scala
def mulOneAtTime(x:Int)(y:Int) = x*y
```
利用柯里化将某个参数单独拎出来，以提供更多用于类型推断的信息
# 控制抽象

 Scala中，可以将一系列语句组成不带参数也没有返回值的函数

```scala
def runInThread(block: () =>Unit){
  new Thread{
    overrride def run(){block()}
  }.start()
}

//调用
runInThread {()=>println("hi")}
```
想要省略掉表示空参数的()，可以这样
```scala
def runInThread(block: =>Unit){
  new Thread{
    overrride def run(){block}
  }.start()
}

//调用
runInThread {println("hi")}
```
这就是控制抽象，看上去像是编程语言关键字的函数。比如我们可以定义一个until函数，作用和while类似
```scala
def until(condition :=> Boolean)(block:=>Unit){
  if(!condition){
    block
    until(condition)(block)
  }
}

//调用
var x =10
until(x==0){
  x-=1
  println(x)
}
```
这样的函数参数叫做换名调用参数，和常规参数不同，函数在调用时参数表达式不会被求值，表达式称为无参函数的函数体，该函数被当成参数传递下去。
until函数时柯里化的，首先处理掉condition，然后把block当成另一个独立参数。

# return表达式

Scala无需用return返回函数值，函数的返回值就是函数体最后的值。但可以用return从匿名函数中返回值给包含这个匿名函数的带名函数。

```scala
def indexof(str:String,ch:Char):Int ={
  var i = 0
  until (i==str.length){
    if(str(i) == ch) return i
    i+=1
  }
  return -1
}
```

匿名函数 if(str(i) == ch) return i;i+=1 的值被传递给until，当return执行时，包含它的带名函数indexof终止并返回给定的值。

如果要在带名函数中使用return，需要给定其返回值类型。

