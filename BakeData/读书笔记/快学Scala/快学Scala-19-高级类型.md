---
title: 快学Scala-19-高级类型
date: 2022-03-01 22:21:35
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# 单例类型

给定任意引用v，可以得到类型v.type，有两个可能的值，v或者null。对于自身进行修改的方法返回this，通过这种方式将方法调用串连起来

```scala
class Document{
  def setTitle(title:String) = {.....;this}
  def setAuthor(author:String) = {......;this}
  ...
}

//调用
article.setTitle("xxx").setAuthor("aaa")
```

对于子类，上述调用方法会存在问题

```scala
class Book extends Document{
  def addChapter(chapter:String) = {...;this}
  ...
}

val book = new Book()
book.setTtile("xxx").addChapter("faf") //报错
```
setTitle方法返回this，而this的类型是Document，没有addChapter方法。此时需要声明setTitle的返回值类型
```scala
def setTitle(title:String):this.type = {.....;this}

// book.setTitle后的类型就是Book了
```
用上述类型定义的手段解决方法串接的问题。
如果要定义一个接受object实例作为参数的方法，也可以直接使用单例类型。

```scala
class Document{
  private var useNextArgAs: Any = null
  def set(obj:Title.type):this.type=(useNextArgAs = obj;this)
  def to(arg:String) = if(useNextArgAs == Title) title=arg; else ...
  ...
}

// Title.type 指代单例对象，不是类型
// set方法
```


# 类型投影

使用嵌套类的时候，两个对象的内部嵌套类是不同的类型，无法将它们添加到同一处。

```scala
class Network{
  class Member(val name:String){
    val contacts = new ArrayBuffer[Member]
  }
  private val members = new ArrayBuffer[Member]
  
  def join(name:string) = {
    val m = new Member(name)
    members+=m
  }
}

//创建两个对象
val chatter = new Network
val myFace = new Network

// chatter.Member 和 myFace.Member 不是同一个类
val fred = chatter.join("Fred")
val barney = myFace.join("Barney")

fred.contacts += barney // 错误，不是同类
```
如果不要这个约束，则可以将Member类直接放在Network类之外，推荐放在Network的伴生对象中。
另一种方式是用类型投影，允许内部类兼容不同对象的内部类

```scala
class Network{
  class Member(val name:String){
    val contacts = new ArrayBuffer[Network#Member]
  }
```

# 路径

如下的表达式称为路径

```scala
com.horstmann.impatient.Network.Member
```
在最后的类型之前，路径的所有组成部分必须是稳定的，必须指定到单个，有穷的范围。组成部分可以是
* 包
* 对象
* val
* this，super，super[S]，c.this
路径组成部分不能是类，即嵌套的内部类不是单个类型，而是给每个实例留出了单独的类型。

同样，类型也不能是var

```scala
var chatter = new Network

val fred = new chatter.Member // 错误，chatter不稳定
```

# 类型别名

对复杂的类型，可以用type关键字来创建一个简单的别名

```scala
class Book{
  import scala.collection.mutable._
  type Index = HashMap[String,[Int,Int]]
  
}
// 可以用Book.Index 代替 
// scala.collection.mutable.HashMap[String,[Int,Int]]
```

# 结构类型

结构类型指一组关于抽象方法，字段和类型的规格说明。这些抽象方法，字段和类型是满足改规格的类型必须具备的。

```scala
def appendLines(target: {def append(str:String):Any},
      lines:Iterable[String]){
        for(l<-lines){target.append(l);target.append("\n")}
      }
      
```
上述函数的参数定义了结构类型，target参数是任何一个定义了append方法的函数，lines是一个支持遍历的类型。
# 复合类型

复合类型写法如下

```scala
T1 with T2 with T3 ...
```
想要成为复合类型，某个值必须满足每一个类型（T1，T2，T3）才行。可以用复合类型来操作必须提供多个特质的值。
# 中置类型

中置类型是一个带有两个类型参数的类型，类型名称写在两个参数中间。

```scala
String Map Int
```
通过多种中置操作符可以定义中置类型
# 存在类型

存在类型是为了和Java的类型通配符兼容，存在类型的定义方式是在类型表达式之后跟上forSome{}，比如

```scala
Array[T] forSome{type T<:JComponent}

Array[_<:JComponent]
```

forSome表示法允许我们使用更复杂的关系，不仅限于类型通配符能表达的那些

```scala
Map[T,U] forSome{type T; type U<:T}
```
forSome 中还可以使用val声明。
```scala
n.Member forSome{ val n: Network }
```

# Scala类型系统

![](1.png)

![](2.png)

# 自身类型

特质可以要求混入它的类扩展自另一个类型，可以用自身类型的声明来定义特质

```scala
trait Logged {
  def log(msg:String)
}

trait LoggedException extends Logged{
  this: Exception =>
    def log(){log(getMessage())}
    // 可以调用getMessage，因为this是一个Exception
}
```

# 依赖注入

在通过组建构建大型系统，而每个组件都有不同实现的时候，我们需要将组件的不同选择组装起来。比如，会有一个模拟数据库和真实数据库，控制台日志和文件日志，某个特定的实现可能想要用真正的数据库和文件日志来运行。要用控制台日志和模拟数据库来测试。

通常，组件之间存在依赖关系，比如，数据访问组件可能会依赖日志功能，Java中Spring框架等工具用来表达这种依赖。每个组件都描述了它所依赖的其他组件的接口。而对实际组件实现的引用，是在程序被组装起来的时候注入的。

Scala中，可以通过特质和自身类型达到一个简单的依赖注入效果。对日志而言

有如下特质和两个实现：ConsoleLogger，FileLogger

认证特质依赖日志功能来记录认真失败

```scala
trait Auth{
  this: Logger =>
    def login(id:String, password:String): Boolean
}
```

应用逻辑依赖认证和日志

```scala
trait App{
  this: Logger with Auth =>
  ...
}
```

可以这样组装应用

```scala
object MyApp extends App 
with FileLogger("test.log") 
with MockAuth("user.txt")
```

更好的方式是通过实例变量来表示组件，而不是将组件粘合成一个大类型。蛋糕模式给出了更好的设计。在这个模式中，对每个服务都提供一个组件特质，该特质包含

* 任何所依赖的组件和自身类型表述
* 描述服务接口的特质
* 一个抽象的val，该val将被初始化成服务的一个实例
* 可以有选择的包含服务接口的实现
```scala
trait LoggerComponent {
  trait Logger {...}
  val logger:Logger
  class FileLogger(file:String) extends Logger{...}
  ...
}

trait AuthComponent{
  this: LoggerComponent =>
  trait Auth{...}
  val auth:Auth
  class MockAuth(file:String) extends Auth{...}
  ...
}

object AppComponents extends LoggerCompponent with AuthComponent{
  val logger = new FileLogger("test.log")
  val auth = new MockAuth("user.txt")
}
```

# 抽象类型

类或特质可以定义在一个子类中被具体化的抽象类型

```scala
trait Reader{
  type Contents
  def read(fileName:String):Contents
}
```
在这里，类型Contents是抽象的，具体的子类需要指定这个类型
```scala
class StringReader extends Reader{
  type Contents = String
  def read(fileName: String) = 
  source.fromFile(fileName,"UTF-8").mkString
}

class ImageReader extends Reader{
  type Contents = BufferedImage
  def read(fileName: String) = ImageIO.read(new File(fileNmae))
}
```

如果类型是在类被实例化时给出的，则使用类型参数。

如果类型是在子类中给出的，则使用抽象类型

# 家族多态

Java中的事件处理，包含不同种类的事件比如ActionEvent，ChangeEvent等，而每种类型都有单独的监听器接口ActionListener，ChangeListener等。这就是家族多态。

对应到Scala中，先定义一个统一的listener

```scala
trait Listener[E]{
  def occurred(e:E):Unit
}

//定义一个监听器的集合和一个触发这些监听器等方法
trait Source[E,L<:Listener[E]]{
  private val listeners = new ArrayBuffer[L]
  def add(l:L){listeners+=l}
  def remove(l:L){listeners-=l}
  def fire(e:E){
    e.source = this
    for(l<-listeners) l.occurred(e)
  }
}
```

接下来可以定义一个带有按钮事件和按钮监听器的按钮时，就可以将定义包含在一个扩展该特质的模块当中：

```scala
object ButtonModule extends ListenerSupport{
  type S = Button
  type E = ButtonEvent
  type L = ButtonListener
  
  class ButtonEvent extends Event
  trait ButtonListener extends Listener
  class Button extends Source{
    def click(){fire(new ButtonEvent)}
  }
}

//使用
object Main{
  import ButtonModule._
  def main(args:Array[String]){
    val b = new Button
    b.add(new ButtonListener{
      override def occurred(e:ButtonEvent){print(e)}
    })
    b.click()
  }
}
```

# 高等类型

泛型类List依赖于类型T并产出一个特定的类型。比如Int，得到的类型是List[Int]。List被称为类型构造器，Scala中，可以定义出依赖于其他类型的类型的类型。（依赖于泛型的类型）。

```scala
trait Iterable[E]{
  def iterator():Iterator[E]
  def map[F](f:(E)=>F):Iterable[F]
}

class BUffer[E] extends Iterable[E]{
  def iterator(): Iterator[E] = ...
  def map[F](f:(E)=>F):Buffer[F] = ...
}
```
对于buffer，我们预期map方法返回一个buffer，不仅仅是Iterable。这意味着不能在Iterable特质中实现这个map方法。可以使用类型构造器来参数话Iterable
```scala
trait Iterable[E,c[_]]{
  def iterator():Iterator[E]
  def build[F](): C[F]
  def map[F](f:(E)=>F):Iterable[F]
}
```
这样以来，Iterable就依赖一个类型构造器来生成结果，以C[_]表示，使得Iterable成为了一个高等类型。
map方法返回的类型可以是也可以不是调用该map方法的原类型。

用range执行map方法，结果可以是另一种类型Buffer[F]

```scala
class Range extends Iterable[Int,Buffer]
```
要实现Iterable中的map方法，Iterable需要能产出包含了任何类型F的值的容器。定义一个Container类：
```scala
trait Container[E]{
  def +=(e:E):Unit
}

trait Iterable[E,C[X]<:COntainer[X]]{
  def build[F]():C[F]
  ...
}
```
类型构造器C现在被限制为一个Container，因此我们知道可以往build方法返回的对象添加项目，不再对参数C使用通配符，因为需要表明C[X]是一个针对同样的X的容器。
之后可以在Iterable特质中实现map方法

```scala
def map[F](f:(E)=>F):C[F]={
  val res = build[F]()
  val iter = iterator()
  while(iter.hasNext) res+=f(iter.next())
  res
}
```
由此，可迭代的类就不再需要提供它们自己的map实现类了。
```scala
class Range(val low:Int, val high:Int)extends Iterable[Int,Buffer]{
  def iterator() = new Iterator[Int]{
    private var i = low
    def hasNext = i <=high
    def next() = {i+=1;i-1}
  }
  
def build[F]() = new BUffer[F]
}
```
Range 只是一个Iterable，可以遍历内容，不能添加，Buffer是Iterable和Container
```scala
class Buffer[E:ClassTag] extends Iterable[E,Buffer] with Container[E]{
  private var capacity = 10
  private var length = 0
  private var elems = new Array[E](capacity)
  
  def iterator() = new Iterator[E]{
    private var i=0
    def hasNext = i<length
    def next() = {i+=1;elems(i-1)}
    }
def build[F:ClassTag]() = new Buffer[F]
def +=(e:E){
  if(length==capacity){
    capacity = 2*capacity
    val nelems = new Array[E](capacity)
    for(i<-0 until length) nelems(i)  =elems(i)
    elems = nelems
  }
  elems(length) = e
  length+=1
}
}
```
上述事例展示了高等类型的典型用例，Iterator依赖Container但Container不是普通的类型，是制作类型的机制。

