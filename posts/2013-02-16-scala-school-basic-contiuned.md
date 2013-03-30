---
layout: post
title: "Scala School--继续基础知识"
permalink: "/2013/scala/scala-school-basic-continue.html"
date: 2013-02-16 
categories: 
- Scala
---
  
_练习题都来至于Twitter的[Scala-School](http://twitter.github.com/scala_school/index.html)_  

### 继续进行基础知识  

##### apply 方法
 `apply`是一种很好的语法糖，当你用类或者对象的其中一种时， 比如 

```
scala> class Foo {}
defined class Foo

scala> object FooMaker {
     |   def apply() = new Foo
     | }
defined module FooMaker

scala> val newFoo = FooMaker()
newFoo: Foo = Foo@5b83f762
```

 相当于调用一个方法式得得到一个对象,或者像调用一个方法式的初始化类;
 
 
```
 scala> class Bar {
     |   def apply() = 0
     | }
defined class Bar

scala> val bar = new Bar
bar: Bar = Bar@47711479

scala> bar()
res8: Int = 0
```
 
### 对象  
 对象通常用于持有类的单一实例。
 
```
 object Timer {
  var count = 0

  def currentCount(): Long = {
    count += 1
    count
  }
}
```  
 你可以这样使用  
 
```
scala> Timer.currentCount()
res0: Long = 1
```  
 
 类和对象通常可以同名。该对象则称呼为`伴生对象`， 下面的例子可以不使用`new`关键字创建类的实例。
 
```
class Bar(foo: String)

object Bar {
  def apply(foo: String) = new Bar(foo)
}
```  

### 函数也是对象  
 Scala中，我们经常谈论的是，我们是在面向函数编程， 那么什么是函数呢？  
 函数就是一组特质的集合， 对于接受一个参数的函数，在Scala中，其是特质`Function1`的实例.  
 
```
scala> object addOne extends Function1[Int, Int] {
     |   def apply(m: Int): Int = m + 1
     | }
defined module addOne

scala> addOne(1)
res2: Int = 2
```

`Function1` 中定义了 `apply` 方法 ，允许你像调用一个函数一样，调用一个对象。  
 在Scala中，类似的函数对象有 `Function` 1~22 . (我们不会想调用一个方法，传入22个参数的)   
  
 并非每次你在类中定义方法，你都会得到一个`Function*`的实例。 方法定义在类中，它仍然是方法，只有在repl中，其才是`Function*`的实例。  
  类也快要继承Function后，仍然以`()`方式来初始化.  
  
  
```
  scala> class AddOne extends Function1[Int, Int] {
     |   def apply(m: Int): Int = m + 1
     | }
defined class AddOne

scala> val plusOne = new AddOne()
plusOne: AddOne = <function1>

scala> plusOne(1)
res0: Int = 2
```  

一种继承`extends Function1[Int, Int]`的简要写法是
  
```
class AddOne extends (Int => Int) {
  def apply(m: Int): Int = m + 1
}
```  
  
### 包  
 
你可以将你的代码组织到某个包中，比如 

```
package com.twitter.example
```  

包和对象往往可以组合起来，很方便的组织静态函数。  

```
package com.twitter.example

object colorHolder {
  val BLUE = "Blue"
  val RED = "Red"
}
```  

访问的时候： 

```
println("the color is: " + com.twitter.example.colorHolder.BLUE)
``` 

(Scala设计者将Scala中的对象设计成Scala的模块系统中的一部分)  

###  模式匹配  

变量模式匹配  

```
val times = 1

times match {
  case 1 => "one"
  case 2 => "two"
  case _ => "some other number"
}
```  

模式匹配的守卫者方式(匹配上后，还要符合后面的守卫条件)  

```
times match {
  case i if i == 1 => "one"
  case i if i == 2 => "two"
  case _ => "some other number"
}
```  

最后的`_`是一个通配符，若没有匹配到，则到它这个分支来了[effective Scala中关于此的记录](http://twitter.github.com/effectivescala/#Functional programming-Pattern matching)  

*类型的模式匹配*  

```
def bigger(o: Any): Any = {
  o match {
    case i: Int if i < 0 => i - 1
    case i: Int => i + 1
    case d: Double if d < 0.0 => d - 0.1
    case d: Double => d + 0.1
    case text: String => text + "s"
  }
}
```  

你也可以利用模式匹配，来匹配类的成员:  

```
def calcType(calc: Calculator) = calc match {
  case calc.brand == "hp" && calc.model == "20B" => "financial"
  case calc.brand == "hp" && calc.model == "48G" => "scientific"
  case calc.brand == "hp" && calc.model == "30B" => "business"
  case _ => "unknown"
}
```  

看上去，还是比较痛苦， 幸亏， Scala提供了一些有用的工具来改善。 比如  

##### Case Classes  


```
scala> case class Calculator(brand: String, model: String)
defined class Calculator

scala> val hp20b = Calculator("hp", "20b")
hp20b: Calculator = Calculator(hp,20b)

```  

`case classes`  自动根据构造函数的参数构建出`equality`和`toString`方法 。
 同样的， case classes 可以像正常的类一样，含有正常的方法. 比如  
 

```
scala> val hp20b = Calculator("hp", "20b")
hp20b: Calculator = Calculator(hp,20b)

scala> val hp20B = Calculator("hp", "20b")
hp20B: Calculator = Calculator(hp,20b)

scala> hp20b == hp20B
res6: Boolean = true
``` 
 
##### case classes with pattern matching

`case classes` 是为了模式匹配而设计的。 比如上面例子，可以使用如下代码   

 
```
val hp20b = Calculator("hp", "20B")
val hp30b = Calculator("hp", "30B")

def calcType(calc: Calculator) = calc match {
  case Calculator("hp", "20B") => "financial"
  case Calculator("hp", "48G") => "scientific"
  case Calculator("hp", "30B") => "business"
  case Calculator(ourBrand, ourModel) => "Calculator: %s %s is of unknown type".format(ourBrand, ourModel)
}
```  
 
 最后的 `_` 可以利用下面的方式来做替换:  
 
```
  case Calculator(_, _) => "Calculator of unknown type"
```  
 
 或者  
 
```
   case _ => "Calculator of unknown type"
```
 
 我们为匹配到的值重新绑定一个名称
  
```
  case c@Calculator(_, _) => "Calculator: %s of unknown type".format(c)
```
  
  
### 异常  
 
 异常是Scala中利用`try-catch-finally`语法来进行的另外一种模式匹配  
 
```
 try {
  remoteCalculatorService.add(1, 2)
} catch {
  case e: ServerIsDownException => log.error(e, "the remote calculator service is unavailble. should have kept your trustry HP.")
} finally {
  remoteCalculatorService.close()
}
```

 `try`可以表达得更加面向表达式一点
 
```
 val result: Int = try {
  remoteCalculatorService.add(1, 2)
} catch {
  case e: ServerIsDownException => {
    log.error(e, "the remote calculator service is unavailble. should have kept your trustry HP.")
    0
  }
} finally {
  remoteCalculatorService.close()
}
```

  