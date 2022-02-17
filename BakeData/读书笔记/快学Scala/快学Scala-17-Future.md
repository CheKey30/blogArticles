---
title: 快学Scala-17-Future
date: 2022-02-17 23:16:45
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 在future中运行任务

scala.concurrent.Future 对象可以在未来执行代码块。

future代码块内的代码会在新线程上执行，并且主线程不会等待其返回结果，当future中的代码完成时，才会组合出结果返回。

```scala
Future{
  Thread.sleep(10000)
  print("run future")
}
print("run current")

// current先打印，然后时future
```

当创建future时，它的代码会在某个线程上执行，一般不是直接创建线程，而是从预先创建了的线程池中获取。Scala中用了ExcutionContext的特质来实现。

每个future在构造时都必须有一个指向某个ExcutionContext的引用，比如

```scala
import ExcutionContext.Implicits.global
```
上述方法会使future全部在一个全局的线程池中运行，如果有大量会阻塞的任务会产生问题。
多个future是并发执行的。总结一句话，Future是一个会在未来某个时间点通知你其中代码成功或者失败的对象。

# 等待结果

可以用isCompleted来检查一个future是否完成。但不用在循环中反复查看是否完成。用一个阻塞的调用来等待结果

```scala
val f = Future{Thread.sleep(10000);42}
val res = Await.result(f,10.seconds)

// 上述代码将等待10s，然后获取结果。如果没有完成，报超时异常
// 如果future还执行失败，再报一个对应的异常

// 也可以分开来写，用ready
Await.ready(f,10.seconds)
val Some(t) = f.value
```
value 方法返回Option[Try[T]]，在future未完成时是None，在future完成时是Some(t)，这里t是Try类的对象，它所持有的是结果或者异常。value方法只有在future完成时才会被调用，因此可以用提取器来获得Try对象。
上述做法其实是阻塞了主线程的，一般不这样处理，回调中展示如何不阻塞进行。

# Try类

Try[T]可能有两种结果，成功或者失败，可以通过match语句来处理try拿到的结果

```scala
t match{
  case Success(v) => print(s"val is $v")
  case Failure(ex) => print(ex.getMessage)
}
```

此外，可以用isSuccess，isFailure 方法获取try对象的结果。

```scala
if(t.isSuccess) print(s"val is ${t.get}")

if(t.isFailure) print(t.failed.get.getMessage)
```

# 回调

为了更好的性能，future应该将结果报告给回调函数 onComplete

```scala
f.onComplete(t => ...)
```
当future完成时，不论成功失败，它会用Try对象来调用给定的回调函数
之后可以根据这个结果作出操作

```scala
val f = Future{
  Thread.sleep(1000)
  if(random()<0.5) throw new Exception
  42
}

f.onComplete{
  case Success(v) => print(s"val is $v")
  case Failure(ex) => print(ex.getMessage)
}
```
如果future任务的计算后跟着另一个计算，就形成了回调嵌套，可行但可读性和代码复杂度太高。
更好的做法是将future当成函数一样可以组合的实体，第一个future的结果传递给第二个。这就是future组合任务。

# 组合future任务

假设向两个web服务发起请求，等待拿到结果，然后将结果拼接起来。通过组合future任务来实现。

```scala
val future1 = Future{getData1()}
val future2 = Future{getData2()}
val combined = future1.map(n1 => future2.map(n2 => n1+n2))

// 当1和2都完成时combined执行
// 返回的结果是Future[Future[Int]]，用flatMap处理
val combined = future1.flatMap(n1 => future2.map(n2 => n1+n2))
```

更加常见的做法是用for yield来对多个future的结果进行串连

```scala
for{
  n1 <- future1
  n2 <- future2 if n1 !=n2
}yield n1+n2
```
简单理解为for中对多个future操作进行处理，并且将返回结果集的每一个元素映射出来（n1，n2）yield中对每一组进行结合操作
for中任意一步失败都会直接终止抛出异常，不必手动添加判断。

如果用def而非val来创建多个future，则能够先后顺序触发，而非并发

```scala
def future1 = Future{getData1()}
def future2 = Future{getData2()}
val combined = future1.map(n1 => future2.map(n2 => n1+n2))

//future2会在future1完成之后才开始求值
//def定义下for中也是有先后顺序的
```

如果第二步依赖第一步，则要用def定义future

# 其他future变换

除了上文提到的map和flatmap，下表是对future内容的其他应用函数

![](1.png)

# future对象中的方法

Future伴生对象包含了用于操作有future组成的集合的若干方法。

对一个集合对象的每一段都进行一个future操作，让其并发执行

```scala
val futures = parts.map(p=>Future{calculate(p)})

// 组合所有future的结果
val res = Future.sequence(futures)
```
如果futures是Set[Future[T]]，则res就是Future[Set[T]]。当futures里每一个future的结果都返回时，res这个future就是一个所有结果组成的集合推进到了完成状态。
一个失败，则全部失败。异常是最左边的那个异常。

# Promise

Future对象是只读的，future的结果在任务结束时隐式的被设置。Promise类似，不过值可以被显示的设置。

```scala
def compute(arg:String){
  val p = Promise[Int]()
  Future{
    val n = workHard(arg)
    p.success(n)
    work()
  }
  p.future
}
```
p.future 交出和promise关联的future，用success或者failure来显示设置值。
# 执行上下文

当任务必须等待时，比如和远程资源通信。有可能会耗尽所有可用线程。此时可以通知执行上下文，报告该线程即将阻塞，方法是把阻塞的代码放入blocking{}中。

```scala
val f = Future{
  val url = ...
  blocking {
    val content = Source.fromURL(url).mkString
  }
}
```

执行上下文收到通知后可能会增加线程数。针对不同的场景可以选择不同的线程池。

