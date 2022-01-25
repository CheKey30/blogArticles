---
title: 快学Scala-7-包和引入
date: 2022-01-24 23:17:09
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 包

和Java相同，包存在的目的是管理大型程序中的名称。

例如，Map这个名字可以有多个定义，如果不用包这种结构进行区分，会造成歧义。

不同包下相同的类名表示完全不同的类。

使用类时，可以用完全限定的名称，即包名.类名或者通过引入包来提供更短的别名。

可以用如下方法添加类到包中

```scala
package com{
  package company{
    package depart{
      class Employee
      ...
    }
  }
}
```
同一个文件中可以有多个上面的包和类的定义。
# 作用域规则

包的嵌套结构中，内层可以访问到外层的包中的内容。

一般不同这种嵌套的方式来引入包

# 串联式包语句

```scala
package com.horstmann.impatient{
  package people{
    class person
  }
}
```
# 文件顶部标记法

常用这种方式标记包，和Java相同，直接在文件顶部不带{}来标记包

只有当文件中的所有代码属于同一个包的时候可以这样，通常也是的

```scala
package com.company.project.beans

class Person{
  
}
```

# 包对象

包可以包含类，对象和特质，但不能包含函数或者变量的定义。Java中工具函数和常量也要添加在Util类里面。

而在Scala中，通过包对象解决这个问题。

每个包都有一个包对象，需要在父包中定义它，且名称和子包一样。

```scala
package com.company.project
// 子包对象
package object subproject{
  val projectName = "subproject1"
}

//获取包对象中的常量
package subproject{
  class Service{
    var name = projectName
  }
}
```
编译时，包对象被编译成带有静态方法和字段的JVM类。
对源文件.scala 也应使用相同的命名规则，包对象也对应一个scala文件，放在包路径下

即在com/company/project/subproject/package.scala

包对象都叫package.scala，放在该包对应的文件夹里。

# 包可见性

Java中，声明为public，private，protected的类在所在包中可见。Scala中也类似

```scala
package com.horstmann.impatient.people

class Person{
// 在包people中函数为private
  private[people] def description = "a persion"
  ...
  
// 在上层包impatient中为private
private[impatient] def description = "a persion"
}
```

# 引入

import，可以用更短的语句调用类，不必写完整类名

```scala
import java.awt.Color

Color // 就是java.awt.Color

//引入包内的全部成员
import java.awt._

//引入类或对象的全部成员
import java.awt.Color._
//Color 中的方法可以直接调用，无需Color.
```

# 任何地方都可以声明引入

Scala中import语句可以出现在任何地方，不止是文件顶部。效果一直持续到包含该语句的语句快的结尾

```scala
class Manager{
  import scala.collection. 
}
```
在需要引入的地方引入，可以减少可能的名称冲突。
# 重命名和隐藏方法

可以用选取器selector来引入包中的几个成员

```scala
import java.awt.{Color,Font}
//重命名引入
import java.util.{HashMap => JavaHashMap}
import scala.collection.mutable._
```

还可以隐藏引用中的成员，避免歧义

```scala
import java.util.{HashMap =>_,_}
import scala.collection.mutable._
// java 中的hashmap被隐藏，程序中调用的都是Scala的hashmap
```

# 隐式引入

每个Scala程序都隐式的以如下代码开始：

```scala
import java.lang._
import scala._
import Predef._
```
对于Scala开头的包，使用是无需再加上Scala前缀。
