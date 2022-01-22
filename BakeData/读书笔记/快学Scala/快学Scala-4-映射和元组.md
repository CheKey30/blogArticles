---
title: 快学Scala-4-映射和元组
date: 2022-01-23 02:39:43
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 构造映射

不可变映射：

```scala
val scores = Map("amy" -> 10, "bob" -> 3, "cindy" -> 8)
```
可变映射：
```scala
val scores = scala.collection.mutable.Map("amy" -> 10, "bob" -> 3,
 "cindy" -> 8)
```
创建空映射需要给定映射实现和值类型
```scala
val scores = scala.collection.mutable.Map[String,Int]()
```

# 获取映射中的值

直接用()表示方法来查询

如果不包括这个键，则抛出异常，contanis方法用来检查是否包括某个键

```scala
val bobScore = scores("bob")
val bobScore = if(socres.contains("bob")) scores("bob")else 0

val bobScore = scores.getOrElse("bob",0)
//类似Java中的getOrDefault
```

# 更新映射中的值

直接用赋值语句更改某个映射的值，如果不存在会创建

通过+=来操作多个

通过-=来移除键

```scala
scores("bob") = 10
scores("frank") = 11
scores+=("bob" -> 10, "frank" -> 7)
scores-="amy"
```
不能更新不可变的映射，但是可以获取一个新映射并且更新旧映射里的值
```scala
val newScores = socres+("bob" -> 10, "frank" -> 7)
//newScores里包含了scores所有的值，并且两条数据被更新了
//其实+=是同样的原理
```

# 迭代映射

scala直接用下面的写法进行映射的迭代操作

for((k,v) <- 映射) 处理 k和v

还可以只访问键或者值

```scala
for((k,v) <- scores) print(k+": "+v)
for(v <- scores.values) print(v)
for(k <- scores.keySet) print(k)

for((k,v) <- scores) yield (v,k)
//反转kv
```

# 已排序映射

映射有两种实现方式，哈希表和平衡树，scala默认用哈希表实现，但返回的元素是乱序的，如果需要顺序，使用sortedMap，会按照key来排序。

还可以使用LinkedHashMap，会按照元素插入顺序排序

# 与Java的互操作

映射可以在Java和Scala间相互转换

Java to Scala:

```scala
import scala.collection.JvavaConversions.mapAsScalaMap

val scores: scala.collection.mutable.Map[String,Int] = 
  new java.util.TreeMap[String,Int]
  

import scala.collection.JavaConversions.propertiesAsScalaMap
val props: scala.collection.Map[String,String] = System.getProperties()
```

Scala to Java:

```scala
import scala.collection.JavaConversions.mapAsJavaMap
import java.awt.font.TextAttribute._//引入下面映射会用到的key
val attrs = Map(FAMILY -> "self", SIZE -> 12) // Scala映射
val font = new java.awt.Font(attrs) // 该方法预期一个Java映射
```

# 元组

元组是不同类型的值的组合，最常见的是对偶元组，映射里的每个元素都是元组。

通过（）将多个单一值扩起来组成元组。

通过_1, _2的形式访问元组中对应的元素

```scala
val t=(1,3.14,"bob")

//上方元组的类型
Tuple3[Int,Double,java.lang.String]

//获取对应的元素
val second = t._2
```

可以用模式匹配的方式获取元组的组成部件

元组用于函数有多个返回值的情况

```scala
val (first,second,third)  = t
val (first,second,_) = t

//将字符串分成两个，大写的拼接成一个，其他的一个
"New York".partition(_.isUpper) //返回("NY", "ew ork")
```

# 
# 拉链操作

通过zip方法，将多个值一起处理，返回多个元组

```scala
val symbols = Array("<","-",">")
val counts = Array(2,10,2)
val pairs = symbols.zip(counts)

//paris是元组组成的数组
Array(("<",2),("-",10),(">",2))

//通过循环处理元组
for((s,n) <- pairs) print(s+" "+n)

/**
toMap 方法将集合转换成映射，
给定key集合和value集合，
通过zip和toMap转换成映射
**/
val newMap = keys.zip(values).toMap
```
