---
title: 快学Scala-2-控制结构和函数
date: 2022-01-23 02:28:31
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 条件表达式

if/else的结构和java相同，不过scala的条件表达式是有值的，可以将其赋值给变量

```scala
val s=if(x>0) 1 else -1

//等效
var s = 0
if(x>0) s=1 else s=-1
```
第一种写法更好，因为第二种s必须是变量
# 语句终止

scala不同于Java，行尾不需要分号表示结束，只有在单行写多个语句才需要分号。

另一方面，如果一个语句过长，一行写不下，则可以在明确不可能是语句结尾的地方换行。比如

```scala
s = s0 +(v1-v2)*t+
  0.5
// 在加号后换行，这两行仍旧属于一条语句
```

# 块表达式和赋值

Java中{}间的语句称为一个块语句，scala中，{} 包含一系列表达式，其结果也是一个表达式，其中最后一个表达式的值就是块的值。

```scala
val dis = {val dx = x-x0; val dy = y-y0; sqrt(dx*dx+dy*dy)}
// dis 的值就是计算出的sqrt的值
```

# 输入输出

print或println或printf都可以打印

打印结果可以用字符串插值来表示，比字符串拼接或者c的printf简单

```scala
print(s"your age is ${age}")

//还可以
print(f"the result is $(res)%7.2f")
//f表示将res按照宽度7，精度2进行省略
//常见的前缀：s，raw，f
```

# 循环

scala的while和java相同，scala没有for(;;)的表达，对应的for写为

```scala
for (i<- 1 to n)
  r = r+i
  
// for(i<- 表达式) 让i遍历表达式的所有值

// 遍历字符串的字符
for (ch<- "hello")
```

# 高级for循环

多层循环直接用多个表达式来表示

```scala
for (i<- 1 to 3; j<- 2 to 4)
  print(f"$(10*i+j)%3d")
  
// 打印 12 13 14 22 23 24 32 33 34
```

通过守卫的形式增加判断，跳过循环中的一些枚举，类似Java中的if() continue;

```scala
for (i<- 1 to 3; j<-1 to 3 if i!=j) print(f"$(10*i+j)%3d")
// 打印12 13 21 23 31 32
// if和前面的表达式不用分号隔开
```

循环中还可以引入变量

```scala
for (i<- 1 to 3; from=4-i; j<- from to 3) print(f"$(10*i+j)%3d")
// 打印 13 22 23 31 32 33
```

如果for循环体以yield开始，则会构造一个集合，每次生成其中一个值

这类循环称为for推倒式

```scala
for (i <- 1 to 3) yield i
// 将返回一个vector(1，2，3)
```

也可以用{}来定义上述表达式

```scala
for {
  i<- 1 to 3
  from = 4 - i
  j<- from to 3
} print(f"$(10*i+j)%3d")
```

# 函数

方法对对象进行操作，而函数则不是，Java中只有静态方法类似函数。c中定义的都是函数

scala中，函数定义如下：

```scala
def abs(x: Double) = if(x>=0) x else -x

// def 开始，函数名(参数名：参数类型) = 函数体
```
上述函数定义没有返回值类型，scala会自动推断，对于递归函数，一定要指定返回值类型
```scala
def fac(n: Int):Int ={
  if (n<=0) 1
  else n*fac(n-1)
}
```

对于没有返回值的函数，返回值类型填Unit

# 默认参数和带名参数

scala函数支持配置默认值，配置后调用时可以不输入对应的参数，取默认值

```scala
def decorate(str:String, left: String="[", right:String="]"):String={
  left + str + right
}

decorate("hello") // 返回 [hello]
```

函数的调用可以用参数名=参数值的形式进行传参，这样不需要保证顺序

# 变长参数

scala允许函数包含不确定的参数个数，即函数可以定义规则对任意个数的传入参数执行这个规则

```scala
def sum(args:Int *) = {
  var res = 0
  for(arg <- args) res+=arg
  result
}

// 求和函数，调用时sum的参数可以是任意多个整数

sum(1 to 5:_*)
// 表示从1到5做参数序列处理
```

# 过程

不返回值的函数被称为过程，就是Java中的void方法。过程有两种定义可以直接省去函数体前的等号，或者用Unit来表明返回值

```scala
def box(s:String){
  print(s)
}

def box(s:String):Unit={
  print(s)
}
```

# 懒值

lazy关键字，用在val的定义时，当val被声明为lazy，则初始化被推迟，直到首次对它取值的时候。

```scala
/** 
此处words是scala中的文件操作，
只有后续真正对该文件进行操作时，words才会初始化，
才会真正打开对应的文件，后续无需再重建
**/
lazy val words = scala.io.Source.fromFile("/usr/share/words.txt").mkString

// 直接打开，不论是否使用
val words = scala.io.Source.fromFile("/usr/share/words.txt").mkString

//每次调用都打开一次
def words = scala.io.Source.fromFile("/usr/share/words.txt").mkString

```


# 异常

和Java一样，当抛出异常时，当前运算终止，系统查找可以接受处理该异常的处理器，如果没有找到，则程序退出。

与Java不同，scala不需要声明函数可能抛出的异常。

throw表达式有特殊的类型Nothing，在if/else中，if/else的类型和另一个分支相同

```scala
if (x>0){
  sqrt(x)
}else throw new IllegalArgumentException("x is negative")

// 表达式的类型是Double
```

具体的异常应该排在通用异常的前面。

# 
