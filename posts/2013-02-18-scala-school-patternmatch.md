---
layout: post
title: "Scala School--模式匹配&函数组合"
permalink: "/2013/scala/scala-school-pattern-match.html"
date: 2013-02-18 
categories: 
- Scala
---

### 函数组合

首先准备两个函数  

```
scala> def addUmm(x: String) = x + " umm"
addUmm: (x: String)String

scala>  def addAhem(x: String) = x + " ahem"
addAhem: (x: String)String
```  

#### compose 
`compose` 可以将其他函数组合起来，形成一个新的函数  ,比如 `g(f(x))` 它会先调用新函数，再调用第二个函数，最后调用第一个函数 

```
scala> val ummThenAhem = addAhem _ compose addUmm _
ummThenAhem: (String) => String = <function1>

scala> ummThenAhem("well")
res0: String = well umm ahem
```

#### andThen

`andThen` 与 `compose` 类似，不同的在于，它会按照顺序首先调用第一个函数(左边的)
 
```
scala> val ahemThenUmm = addAhem _ andThen addUmm _
ahemThenUmm: (String) => String = <function1>

scala> ahemThenUmm("well")
res1: String = well ahem umm
```  

### PartialFunction  

 通常来说，一个函数的每一个参数类型都是定义好的。 比如 `(Int)=>String` 其接受任意的`Int`值，返回一个`String` 对象   
 而`PartialFunction` 则可能只接受定义好的参数类型的某一些值。 比如 PartialFunction 函数`(Int)=>String` 可能并不接受每一个`Int`值.  `isDefinedAt`方法可以决定是否接受传入的参数 .  
 
```
val one: PartialFunction[Int, String] = { case 1 => "one" }
one: PartialFunction[Int,String] = <function1>

scala> one.isDefinedAt(1)
res0: Boolean = true

scala> one.isDefinedAt(2)
res1: Boolean = false
```  

上面的例子中，只有传入`1`才会被函数所接受;    

`PartialFunction` 可以利用 `orElse` 关键字来进行多个组合.  

```
scala> val two: PartialFunction[Int, String] = { case 2 => "two" }
two: PartialFunction[Int,String] = <function1>

scala> val three: PartialFunction[Int, String] = { case 3 => "three" }
three: PartialFunction[Int,String] = <function1>

scala> val wildcard: PartialFunction[Int, String] = { case _ => "something else" }
wildcard: PartialFunction[Int,String] = <function1>

scala> val partial = one orElse two orElse three orElse wildcard
partial: PartialFunction[Int,String] = <function1>

scala> partial(5)
res24: String = something else

scala> partial(3)
res25: String = three

scala> partial(2)
res26: String = two

scala> partial(1)
res27: String = one

scala> partial(0)
res28: String = something else
``` 

由于`PartialFunction` 是`Function`的一个子类型，所以 集合的 `filter`方法中，也可以接受一个`PartialFunction` , 比如  

```
scala> case class PhoneExt(name: String, ext: Int)
defined class PhoneExt

scala> val extensions = List(PhoneExt("steve", 100), PhoneExt("robey", 200))
extensions: List[PhoneExt] = List(PhoneExt(steve,100), PhoneExt(robey,200))

scala> extensions.filter { case PhoneExt(name, extension) => extension < 200 }
res0: List[PhoneExt] = List(PhoneExt(steve,100))
```  


其他有关的信息可以在这里找到: [Effective Scala#PartialFunction](http://twitter.github.com/effectivescala/#Functional programming-Partial functions)  


