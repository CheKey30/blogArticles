---
title: 快学Scala-13-集合
date: 2022-02-10 23:56:39
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 主要的集合特质

![](1.png)

上图表示了Scala中集合类继承关系中的关键特质

Iterable指能够提供一个遍历集合中每个元素的方法

seq是有先后次序的值的序列，比如数组和列表

IndexedSeq允许我们通过整型下标快速访问任意元素

Set是没有先后顺序的集合

SortedSet中元素以某种顺序被访问

Map是一组一组元素的集合

SortedMap是按照键的某种顺序访问其中的实体

每个Scala集合特质或类都有一个带有apply方法的伴生对象，这个apply方法可以用来构建该集合中的实例

```scala
Iterable(0xFF,0xFF00)
Set(1,2,3)
Map(1->"A",2->"B",3->"C")
SortedSet("hello","world")
```
这样的设计叫做统一创建原则
不同集合间可以相互转换，常见方法有toSeq，toSet，toMap等，还有范型方法to[C]

```scala
val coll = Seq(1,2,4,5)
val set = coll.toSet
val buffer = coll.to[ArrayBuffer]
```

同类集合间进行比较可以用===，是值的比较，不通类型集合间比较，需要用sameElements方法

# 可变和不可变集合

Scala支持可变和不可变两种集合，不可变集合从不改变，可以安全的共享引用，多线程中也没有问题。

scala.collection.mutable.map 和 scala.collection.immutable.map 有共同的超类scala.collection.map。分别对应可变和不可变的map

Scala优先采用不可变集合，

```scala
scala.collection.map("hello"->2)//是一个不可变的map

import scala.collection.mutable
val m1 = Map(1->3) // 不可变map
val m2 = mutable.Map(1->3) //可变map
```
不可变集类似于Java中的String，每次对其修改都是产生了一个新的集，不同点在于，如果新集中加入了老集已有的元素，则这个元素是引用。
# 序列

![](2.png)

上图展示了Scala中的不可变序列

Vector是ArrayBuffer的不可变版本，支持快速随机访问。向量是以树形结构（类似b+树）的形式实现的。

Range表示一个整数序列，range对象不存储所有值，只存储开始，结束和增值。用to和until方法来构造。

![](3.png)

上图展示了Scala中的可变序列。

# 列表

在Scala中，列表类似数据结构链表，列表要么是Nil，空表。要么是提供head元素加一个tail，而tail又是一个列表

```scala
val digits = List(4,2)
/**
digits.head 的值是4，digits.tail是List(2)
进一步说，digits.tail.head是2，digits.tail.tail是Nil
**/
```

::操作符从给定的头和尾创建一个新的列表

```scala
9::List(4,2)
//就是List(9,4,2)
9::4::2:Nil
//::是右结合的，从尾开始创建列表
```

除了用迭代器遍历链表，Scala中使用递归更好

```scala
def sum(lst:List[Int]):Int =
{
  if(lst==Nil) 0 else lst.head + sum(lst.tail)
}
```

还可以用模式匹配来遍历

```scala
def sum(lst:List[Int]):Int = 
{
  case Nil =>0
  case h::t => h+sum(t)
}
```

对于sum，Scala已经提供了直接list.sum即可

如果想修改可变列表里的元素，可以用ListBuffer，这是一个由链表支持的数据结构，包含一个指向最后一个节点的引用。因此可以高效的从任意一端增减元素。

改变列表元素是低效的，最佳选择是用结果生成一个新的列表。

# 集

集是不重复元素的集合，尝试将已有元素加入是无效的。

集不保留元素的插入顺序，默认情况下，集以哈希集实现。元素根据hashCode的值进行组织。Scala和Java一样，每个对象都有hashCode方法。

链式哈希集可以记住元素被插入的顺序，它会维护一个链表来达到这个目的

```scala
val weekdays = scala.collection.mutable.LinkedHashSet(
"Mo","TU","We","TH","FR")

//sorted set会根据元素进行排序
val nums = scala.collection.mutable.SortedSet(1,2,3,4)
```

位组是集的一种特殊实现，用一个字位序列的形式存放非负整数，如果位组中第i位是1，说明i在这个集中。Scala提供了可变和不可变两种BitSet

contains方法检查元素是否存在，subsetOf方法检查某个集是否是另一个的子集。

union，intersect和diff方法也有

```scala
val digits = Set(1,2,34)
digits contains 0
Set(1,2) subsetOf digits
val p = Set(4,5,6)

digits union p // digits | p
digits intersect p // digits & p
digits diff p // digits &~ p
```
# 用于添加或去除元素的操作符

不同类型的集合对应不同的操作符

![](4.png)

+用于将元素添加到无先后次序的集合

+:和:+ 用于添加到有序集合到头尾

这些操作符都返回新集合

可变集合有+=来进行操作

不可变集合可以在var上使用+=或:+=

```scala
var num = Set(1,2,4)
num+=5 //将num设置为不可变集 num+5
var numV = Vector(1,2,3)
numV :+=5 // 向量没有+操作符
```

# 常用方法

Iterable特质常用的方法如下：

![](5.png)

![](6.png)

seq多出了如下常用方法：

![](7.png)

![](8.png)

这些方法不改变原有集合，它们返回一个和原有集合相同类型的新集合，这被称为统一返回类型原则。

# 将函数映射到集合

对集合中的所有元素进行操作，可以用map方法

```scala
val names = List("a","b","c")
names.map(_.toUpperCase) //转大写
```
如果传入的映射函数是一个输入，多个输出，则可以用faltMap将结果转为一个集合
```scala
val str = List("a,c,d,e","b,e,f,g")
str.flatMap(_.split(","))
```

transform方法是map的等效操作，只不过当场执行而不是交出新的集合。应用于可变集合，将每个元素替换为转换后的。

groupBy方法交出一个映射，键是函数求值后的值，值是函数求值得到给定键的元素的集合。

```scala
val words = List("Afc","Cff","Bfa","Add")
val map = words.groupBy(_.substring(0,1).toUpper)
/**
得到一个map，key是单词的首字母，value为这个首字母所对应的所有单词
A:Add,Afc
B:Bfa
C:Cff
**/
```
# 化简、折叠和扫描

用二元函数来组合集合中的元素，如

```scala
List(1,2,4,5).reduceLeft(_-_)
//得到((1-2)-4)-5

List(1,2,3,5).reduceRight(_-_)
//得到(1-(2-(3-5)))

List(1,3,4,5).foldLeft(0)(_-_)
//得到0-1-3-4-5
```

利用这种二元函数的操作，可以实现一些统计的功能，比如统计集合中元素出现的频率，这就是折叠

```scala
(Map[Char,Int]()/:"fasdvapdsf"){
  (m,c) => m+(c->m.getOrElse(c,0)+1)
}
```
# 拉链操作

当存在两个集合，想将其中对应的元素结合在一起时，可以用拉链操作

比如将商品和价格两个集合结合起来

```scala
val prices = List(5,4,3,1)
val goodIds = List(1,2,3,4)

prices zip goodIds
//得到list((5，1),(4,2)...)
```
之后可以调用map等函数对每一个数对进行操作
如果两个集合长度不同，zip结果中舍弃长集合多出来的值

zipAll方法可以指定短集合的默认值

# 迭代器

对于之前map等操作无法满足的场景，也可以使用迭代器调用next方法获取每一个元素

```scala
while(iter.hasNext){
  iter.next()
}

for(elem <- iter){
  elem
}
```
如果想查看下一个元素，又不想移动迭代器指针，用buffered方法转化迭代器成为BufferedIterator，head方法用于获取下一个元素但不移动指针。
# 流

流是一个尾部被懒计算的不可变列表，只有当需要时才会被计算

```scala
def numsForm(n:BigInt):Strem[BigInt] = n #::numsFrom(n+1)

val tenOrMore = numsFrom(10)
tenOrMore.tail.tail
//每次tail调用才会计算下一个元素
```

# 懒视图

对集合应用view方法实现懒加载，该方法交出一个总是被懒执行的集合

```scala
val p = (1 to 10000).view
.map(x=>x*x)
.filter(x=>x.toString == x.toString.reverse)

p.take(10).mkString(",")
```

# 与Java集合的互操作

JavaConversions 对象提供了Scala和java集合之间转换的一组方法

![](9.png)

![](10.png)


# 并行集合

scala 对并发求值提供了很好的支持，类似mapreduce，通过并行的操作提高效率。

```scala
coll.par.sum
//par方法是一个并行的具体实现，coll是被操作的集合

for(i<-(0 until 10000).par) print(s"$i")
//循环可以并行化执行

val result = coll.par.filter(p).seq
//并行结果转化回顺序序列
```
