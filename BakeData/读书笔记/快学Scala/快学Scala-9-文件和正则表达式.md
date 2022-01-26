---
title: 快学Scala-9-文件和正则表达式
date: 2022-01-26 23:02:27
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 读取行

通过scala.io.Source对象的getLines方法可以读取文件中的所有行

```scala
import scala.io.Source
val source = Source.fromFile("myfile.txt","UTF-8")
val lineIterator = source.getLines
for(l <- lineIterator)
  print(l)

// 按行读取成数组  
val lines = source.getLines.toArray

// 读取成一个字符串
val contents = source.mkString

//记得关闭
source.close()
```

# 读取字符

读取字符，可以直接把source当迭代器，因为source扩展自Iterator[Char]

```scala
for(c <- source)
  print(c)
  
```
通过source的buffered方法，可以查看某个字符但不把它当做已经处理过的字符（指针不往下走）
```scala
val source = Source.fromFile("myfile.txt","UTF-8")
val iter = source.buffered
while(iter.hasNext){
  if(iter.head =='$'){
    iter.head = '#'
    iter.next
  }
}
source.close()
//替换所有的$
```

# 读取词法单元和数字

直接按照空格分隔成一个数组

```scala
val tokens = source.mkString.split("\\s+")

//转换成数字
val numbers = for(w<-tokens) yeild w.toDouble
```

# 从URL或其他源读取

Source 还支持非文件的源，比如url

```scala
val source1 = Source.fromURL("http://www.aa.com/index.txt","UTF-8")
val source2 = Source.fromString("hello world")
val source3 = Source.stdin
```

# 读取二进制文件

需要使用Java读取二进制文件的类库

```scala
val file = new File(filename)
val in = new FileInputStream(file)
val bytes = new  Array[Byte](file.length.toInt)
in.read(bytes)
in.close()
```

# 写入文本文件

写入同样需要调用java的类库

```scala
val out = new PrintWriter("numbers.txt")
for(i<-1 to 100) out.println(i)
out.close()
```

# 访问目录

同样需要使用Java的类库

```scala
import java.nio.file_
String dirname = "/user/local/test"
val entires = Files.walk(Paths.get(dirname))
try{
  entires.forEach(p => process the path p)
}finally {
  entires.close()
}
```

# 序列化

Java中被序列化的类需要实现Serializable接口

Scala集合类都是可以序列化的，因此序列化类中可以包括集合类的属性

```scala
class Person extends Serializable{
  private vla friends = enw ArrayBuffer[Person]
  ...
}
```

# 进程控制

scala.sys.process包提供了用于和shell程序交互的工具，可以用scala编写shell脚本，同时充分利用scala的其他特性

```scala
import scala.sys.process._
"ls -al ..".!
//这行的执行结果是显示上层目录的所有文件结果打印到标准输出
//!执行的是ProcessBuilder对象，这里有字符串到ProcessBuilder的隐式转换
```
# 正则表达式

scala.util.matching.Regex类用于操作正则表达式，构造一个正则表达式直接用string类带的r方法

```scala
val numPatten = "[0-9]+".r
```
推荐使用"""..."""字符串来写正则表达式，这样不用处理特殊斜杠的转义
findAllIn方法返回所有匹配项的迭代器，直接用for循环操作返回结果，或者转换为数组

```scala
val pattern = """\s+[0-9]+\s+""".r
for(matching <-pattern.finallIn("99 is 100, are 1000"))
  print(matching)

val matches = pattern.findall("99 1000, are 1030").toArray
```

一系列函数用于替换正则匹配到的字符串

```scala
pattern.replaceFirstIn("103 2032")
pattern.replaceAllIn("134234")
pattern.replaceSomeIn("f234f23rrqw", m=> if (m.matched.toInt%2 = 0))
```

# 正则表达式组

在想要提取子表达式两侧加上圆括号，可以对表达式分组

```scala
val patterns = "([0-9]+) ([a-z]+)".r

patterns.findAllMatchIn("01Ab 03Cd")
for(matching <-patterns){
  println(matching.group(1)+" "+matching.group(2))
}
/**
这里返回的迭代patterns中，每个元素都有两个分组，
分别对应正则中两处匹配，即返回的不是简单的string，而是多元组
这里打印结果为：
01 ab
03 Cd
**/
```

