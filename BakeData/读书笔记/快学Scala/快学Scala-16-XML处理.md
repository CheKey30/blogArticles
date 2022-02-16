---
title: 快学Scala-16-XML处理
date: 2022-02-16 19:53:50
categories:
- 读书笔记
- 快学Scala
tags:
- scala
---


# XML字面量

 Scala支持CML字面量，直接用XML代码即可

```scala
val doc = <html><head><title>AAA</title></head><body>...</body></html>
```

doc 的类型是scala.xml.Elem，XML字面量也可以是一系列的节点

```scala
val items = <li>Fred</li><li>Willam</li>
```
items的类型是scala.xml.NodeSeq
# XML节点

Node类是所有XML节点类型的祖先，两个最重要的子类是Text和Elem。Elem类描述的是XML元素。elem包括一个label属性保存标签名称，child属性是嵌套的子元素。

NodeSeq是Seq[Node]的子类，加入了对XPath操作的支持。

XML相关节点类继承关系如下：

![](1.png)

可以通过NodeBuffer来逐渐构建节点序列。

```scala
val items = new NodeBuffer
items += <li>Fred</li>
items += <li>Willam</li>
```

# 元素属性

通过attributes属性来处理某个元素的属性键和值

```scala
val elem  = <a href = "http....">Link</a>
val url = elem.attributes("href")

// 用text将结果转换为字符串
val strUrl = elem.attributes("href").text

// 可以添加getOrElse处理空值
val url = elem.attributes.get("href").getOrElse(Text(""))

// 可以直接遍历
for(att <- elem.attributes)
  print(att.key + att.value.text)
  
```

# 内嵌表达式

可以在XML字面量中包含Scala代码块，动态计算出元素内容

```scala
<li>{items(0)}</li><li>{items(1)}</li>
```

XML内嵌的Scala代码中还可以继续包含XML字面量

```scala
<ul>{for (i<- items) yield <li>{i}</li>}<ul>
```

# 在属性中使用表达式

可以用Scala表达式来计算属性值

```scala
<img src = {makeURL(fileName)}>
// ""内包括的内容直接展示为字符串，不是函数
```

如果内嵌代码块返回null或None，则该属性不会被设置

```scala
<img alt = {if(desc =="TODO")null else desc}.../>

//如果为null，则没有alt这个属性
```

# 特殊节点类型

如需将非XML文本包含到XML文档中，比如将XHTML页面中的JavaScript代码嵌入，可以在XML字面量中使用CDATA标记

```scala
val js = <script><![CDATA[if(temp<0) alert("Cold")]]></script>

//解析出的CDATA会是下面这样的一个text节点
val code = """if(temp<0) alert("Cold")"""

//这样会解析成一个PCData节点
val js = <script>{PCData(code)}</script>
```

# 类XPath表达式

NodeSeq类提供了类似于XPath中/和//操作符的方法。Scala中//是注释，因此这里用\\和\代替。

\定位某个节点或节点序列的直接后代

```scala
val list = <dl><dt>Java</dt><dd>gisling</dd><dt>Scala</dt><dd>odera</dd></dl>

val a = list\"dt"

// a是所有dt标签里的内容，此处是Java和Scala
```

# 模式匹配

模式匹配支持XML字面量

```scala
node match{
  case <img/> => ...
  //匹配单个后代
  case <li>{_}</li> =>
  //匹配任意多项
  case <li>{_*}</li> =>
  //使用变量名来绑定内容
  case <li>{child}</li> => child.text
  //匹配文本节点
  case <li>{Text(item)}</li> => item
  //绑定序列到变量
  case <li>{childern @ _*}</li> => for(c <- children) yield c
}
```

# 修改元素和属性

 XML节点序列是不可变的，想要编辑，需要创建拷贝，给出修改。

```scala
val list = <ul><li>Fed</li><li>Willim</li></ul>

// 将ul标签改成了o1
val list2 = list.copy(label  = "o1")

//添加后代
val list3 = list.copy(child = list.child ++ <li>another</li>)

//改变属性
val img = <img src = "aaa.jpg">
val img2 = img % Attribute(null,"alt", "an img", Null)
```

# XML变换

 XML类库的RuleTransformer类，可以批量更改满足某个特定条件的后代。具体实现是将RewriteRule实例应用在节点及其后代上。

```scala
//创建一个RewriteRule，重写transform方法
val rule1 = new RewriteRule {
  override def transform(n:Node) = n match{
    case e @ <ul>{_ *}</ul> => e.asInstanceOf[Elem]copy(label= "o1")
    case _ => n
  }
}

//应用规则到XML上
val transed = new RuleTransformer(rule1).transform(root)
```

# 加载和保存

 XML对象的loadFile方法用于从文件中加载XML

```scala
import scala.xml.XML
val root = XML.loadFile("myFIle.xml")
```

保留用save方法

```scala
XML.save("myFile.xml",root)
```

# 命名空间

类似Java和scala中的包概念，命名空间在XML避免名称冲突，XML的命名空间是一个URI。

xmlns属性用于声明命名空间

```xml
<html xmlns = "http....">.....</html>
```
 Scala中，每个元素都有一个scope属性，该类的uri就是XML的命名空间。
多个命名空间也可以用前缀的形式进行嵌套，scope.parent对多个命名空间进行串联。

