---
title: 快学Scala-3-数组相关操作
date: 2022-01-23 02:39:21
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 定长数组

 Scala中的Array是定长数组

```scala
val nums = new Array[Int](10)
//定长的整数数组，长度为10，初始值为0

val s = Array("hello","word")
//直接给定数组元素，不需要new

s(0) = "google"
// 给第0个元素赋值
```
scala 中的数组以Java数组的方式实现
# 变长数组：数组缓冲

Java中有ArrayList，Scala中有ArrayBuffer

```scala
import scala.collection.mutable.ArrayBuffer
val b = ArrayBuffer[Int]()
b+=1
//在数组末尾加一个元素1
b+=(1,2,3,4)
//在末尾增加4个元素
b++=Array(8,13,21)
//通过++=来追加任任何集合
b.trimEnd(5)
//移除最后5个元素
b.insert(2,6)
//在下标2插入数字6
b.insert(2,7,8,9)
//在下标2插入多个元素
b.remove(2)
//移除下标2处的元素
b.remove(2,5)
//移除下标2开始后续5个元素
b.toArray
//将缓存b转换为数组，意味着长度已经定下，不变了
b.toBuffer
//将数组转换成缓存，可以继续改动
```

# 遍历数组和数组缓冲

scala中两种数据结构可以用同样的代码处理遍历

```scala
for (i <- 0 until a.length)
  print(a(i))
  
// a 是一个数组或者数组缓存
```
until和to类似，但是until会排出最后一个元素，因此这里不需要length-1，不会outOfRange
还可以通过by来控制遍历的顺序和条件

```scala
for(i <- 0 until a.length by 2)
// 0,2,4,6,8 ...

for(i <- 0 until a.length by -1)
// 逆序遍历
```

还可以不用数组下标，直接遍历元素

```scala
for(ele <- a)
// 遍历a中的每一个元素
```

# 数组转换

scala允许对数组进行转换，转换是指对数组进行一定操作生成一个新的数组而不改变原来的数组。

```scala
val a = Array(1,2,3,5)
val result = for(ele <-a) yield 2*ele

// result 是 (2，4，6，10)的新数组
```

for/yield创造一个相同类型的集合，结果包含yield后的表达式，每次迭代对应一个。

# 常用算法

数组可以直接调用一些常见的方法，比如sum，max，min，sorted，sortWith(_>_)，mkString等

```scala
val a = Array(1,2,3,4)
a.sum
a.max
a.sorted
a.sortWith(_>_)
a.mkStirng("and")
//以"and"分隔所有a中元素输出
```

# 解读Scaladoc

一些常见的数组操作函数

![](1.png)

![](2.png)


# 多维数组

多维数组通过ofDim来构建，通过两对括号来访问

```scala
val matrix = Array.ofDim[Double](3,4)
// 创建一个三行四列的数组

matrix(0)(3) // 获取0行3列
```

也可以指定不规则多维数组，每行的长度不一样

```scala
val tr = new Array[Array[Int]](10)
for(i <- tr.indices)
  tr(i) = new Array[Int](i+1)
  
```

# 与Java的互操作

数组类型匹配的情况下，scala和Java的数组可以相互传递。

```scala
val command = ArrayBuffer(1,2,3)
val pb = new ProcessBuilder(command) // Scala 转 Java
val cmd: Buffer[Int] = pb.command() // Java转Scala
```


