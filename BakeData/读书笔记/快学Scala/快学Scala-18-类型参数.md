---
title: 快学Scala-18-类型参数
date: 2022-02-18 20:20:15
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 泛型类

Scala中可以用类型参数来实现类和函数，这样的类和函数可以用于多种类型。比如Array[T]可以存放任意类型T的元素。

Scala中用方括号定义类型参数

```scala
class Pair[T,S](val fisrt:T, val second: S)
```
以上定义一个带两个类型参数T和S的类。在类的定义中，可以用类型参数来定义变量，方法，参数和返回值类型。使用时直接指定两个类型，比如Pair[Int, String]。更简单一点，Scala会推断对应的类型
```scala
val p = new Pair(31,"aaa")
```

# 泛型函数

函数和方法也可以带类型参数

```scala
def getMiddle[T](a:Array[T]) = a(a.length/2)
```
 Scala会推断实际类型，也可以手动定义
```scala
getMiddle(Array(1,2,3,4,5))

val f = getMiddle[Int] _ // 将指定类型后的函数保存到f
```

# 类型变量界定

考虑一个pair类型，并且对其定义一个取小方法

```scala
class Pair[T](val first: T, val second: T){
  def smaller = if(first.compareTo(second)<0)first else second
}
```
上述代码是错误的，因为不知道T类型是否有compareTo方法。因此，要对T类型进行限制，称为对T添加一个上界。
```scala
class Pair[T <: Compareable[T]](val first: T, val second: T){
  def smaller = if(first.compareTo(second)<0)first else second
}
```

类似的，还可以对类型指定下界。比如对pair定义一个替换第一个元素的函数

```scala
class Pair[T](val first: T, val second: T){
  def replaceFirst(newFirst: T) = new Pair[T](newFirst,second)
}
```
对上述方法进一步扩展，对于first的替换，newFirst的类型可以是T，也可以是T的超类。这种情况需要定义变量边界
```scala
class Pair[R >:T]
```

# 视图界定

视图界定是为了解决继承关系不同导致的某些函数未被实现的情况比如Int没有compareTo，可以改为这样

```scala
class Pair[T <% CompareTo[T]]
```
但视图界定已经过时，可以用类型约束替换。
```scala
class Pair[T](val first: T, val second: T)(implicit ev:t =>Comparable[T]){
  ...
}
```

# 上下文界定

上下文界定的形式是 T:M，M是另一个泛型类，它要求必须存在一个类型为M[T] 的隐式值。将在21章介绍。

# ClassTag上下文界定

要实例化一个泛型的Array[T]，需要一个ClasssTag[T]对象

```scala
def makePair[T:classTag]{first:T, second: T} = {
  val r = new Array[T](2)
  r(0) = first
  r(1) = second
}
```

当调用makePair(4,9) 编译器将定位到隐式的classTag[Int]并实际上调用makePair(4,9)(classTag)。

在虚拟机中，泛型相关的信息是被抹掉的，这时只会有一个makePair方法，却要处理所有类型T。

# 多重界定

类型变量可以同时有上界和下界。不能有多个上界或者下界，但能实现多个特质。

```scala
T <:Comparable[T] with Serializable with Cloneable
```

# 类型约束

类型约束提供另一种限定类型的方式。

```scala
T =:= U
T <:< U
T => U
```
上述约束分别测试T是否等于U，是否U的子类，能否转换为U。通过添加隐式类型证明参数来使用。
```scala
class Pair[T](val first: T, val second: T)(implicit ev: T <:< Comparable[T])
```

类型约束可以帮助定义出只有特定条件下才能用的方法

```scala
class Pair[T <: Compareable[T]](val first: T, val second: T){
  def smaller(implicit ev: T<:<Ordered[T]) = 
    if (first<second) first else second
}
```
只有T可以排序才能调用smaller方法
类型约束还可以改进类型推断

```scala
def firstLast[A,C <:Iterable[A]] (it:C) = (it.head,it.last)
```
执行firstLast(List(1,2,3))
会得到报错，[Nothing, List[Int}]  不符合 [A,C <:Iterable[A]]。此时需要用类型约束来指定先匹配C，再匹配A

```scala
def firstLast[A,C](it:C)(implicit ev: C <:< Iterable[A]) = (it.head,it.last)
```

# 型变

假设一个函数对pair[Person]进行处理，能否将pair[Child]当作参数传入呢？虽然Child是Person的子类，但默认情况下不行。

需要在定义Pair类时表名这一点才可以

```scala
class Pair[+T](val first: T, val Second T)
```
+表示该类型和T协变，接受T的子类。
-表示该类型和T逆变，接受T的超类。

# 协变和逆变点

如果一个对象同时消费和产出某值，则类型应该保持不变。比如数组就不支持型变。Array[Child]和Array[Person]不能相互转换。

协变点和逆变点指将子类传入的点和将超类传入的点。

# 对象不能泛型

无法给对象添加类型参数，比如可变列表，元素类型为T的列表要么空，要么是一个头部类型为T，尾部类型为List[T]的节点。这里T必须是确定的一个，不能是泛型。

可以用继承Nothing的方式来对对象传入类型参数，因为Noting是所有类型的子类，这里用协变解决。

```scala
object Empty extends List[Noting]
val lst = new Node(34,Empty)
```

# 类型通配符

 Java中，所有泛型类都是不变的，不过可以用通配符改变类型。

```java
void makeFriends(List<? extends Person> people)

//可以用 List<Student> 作参数调用
```

Scala中对于协变或者逆变的Pair类，无需使用通配符，但假定pair是不变的，可以定义

```scala
def makeFriends(p:Pair[_ <:Person]) 

def min[T](p:Pair[T])(comp: Comparator[_ >:T])
```

