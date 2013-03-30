---
layout: post
title: "Scala School--集合"
permalink: "/2013/scala/scala-school-collection.html"
date: 2013-02-17 
categories: 
- Scala
---

### 集合

_练习题都来至于Twitter的[Scala-School](http://twitter.github.com/scala_school/index.html)_  


#### 基本数据结构  
Scala提供了一些非常有用得集合，你可以参见[Effective Scala](http://twitter.github.com/effectivescala/#Collections)  
  
##### Lists  

```
scala> val numbers = List(1, 2, 3, 4)
numbers: List[Int] = List(1, 2, 3, 4)
```

##### Tuple(元组)
 
元组可以对一些不同类型的数据进行组合.

``` 
scala> val hostPort = ("localhost", 80)
hostPort: (String, Int) = (localhost, 80)
```    

与 `case classes` 不同的是 ,元组没有名称，也无法通过名字来进行访问。 它们通过下标来进行访问， 从 1 开始  

```
scala> hostPort._1
res0: String = localhost

scala> hostPort._2
res1: Int = 80
```  

元组可以很方便的进行模式匹配  

```
hostPort match {
  case ("localhost", port) => ...
  case (host, port) => ...
}
```

可以使用`->`操作符号来用简易语法构建元组  

```
scala> 1 -> 2
res0: (Int, Int) = (1,2)
```  


##### Maps 
 Maps能持有一些基本数据类型  
 
```
Map(1 -> 2)
Map("foo" -> "bar")
```  

其中的`->`与之前讨论的创建元组的语法有相同之处， Maps其内部的值可以持有Maps或者函数  

```
Map(1 -> Map("foo" -> "bar")) 
Map("timesTwo" -> { timesTwo(_) })
``` 

##### Option  
`Option` 表示一种可能存在又或可能不存在的值.   
`Option `接口的形式如下:  

```
trait Option[T] { def isDefined: Boolean def get: T def getOrElse(t: T): T }
```  

`Option`它有两个子类 `Some[T]` 和 `None`  ，来看一个实例:  


```
scala> val numbers = Map(1 -> "one", 2 -> "two")
numbers: scala.collection.immutable.Map[Int,String] = Map((1,one), (2,two))

scala> numbers.get(2)
res0: Option[java.lang.String] = Some(two)

scala> numbers.get(3)
res1: Option[java.lang.String] = None
```  

 但是一般来说，建议使用 `getOrElse` 方法来获取这种可能不存在的值  
 
```
val result = res1.getOrElse(0) * 2
```  

而`Option`类型可以很自然的用在模式匹配  

```
val result = res1 match { case Some(n) => n * 2 case None => 0 }
```  

你可以在[Effective Scala#Option](http://twitter.github.com/effectivescala/#Functional programming-Options) 中看到更多关于Options的资料  


  
 
### 常用的集合函数组合  

##### map操作  
 遍历集合中的元素，并应用与集合的每一个元素， 比如下面的实例:  

```
scala> val numbers=List(1,2,3,4)
numbers: List[Int] = List(1, 2, 3, 4)

scala> numbers.map((i:Int)=> i*2)
res0: List[Int] = List(2, 4, 6, 8)
```  

或者，直接将函数传入到map方法内部， 比如  

```
scala> def timesTwo(i:Int):Int = i*2
timesTwo: (i: Int)Int

scala> numbers.map(timesTwo _)
res1: List[Int] = List(2, 4, 6, 8)
```  

##### foreach 
 与`map`函数类似，但是没有返回值 , 当然，你也快要尝试将`foreach`的操作结果存储起来，但是其类型为 `Unit` (类似于`void`)  
 
```
scala> numbers.foreach((i: Int) => i * 2)

scala> val doubled = numbers.foreach((i: Int) => i * 2)
doubled: Unit = ()
```   

##### filter  
 根据指定的函数，来过滤集合中的元素 ，比如，要查找元素所符合某个条件的值  
 
```
scala> numbers.filter((i:Int) => i==1 )
res2: List[Int] = List(1)
```  
或者直接使用函数

```
scala> def isOne(i:Int):Boolean = i==1
isOne: (i: Int)Boolean

scala> numbers.filter(isOne _)
res3: List[Int] = List(1)
```  

##### zip 
  这个又称之为`拉链`操作， 把相同长度的两个集合组合起来，形成键值对形式的新集合.  
  
```
scala> List(1, 2, 3).zip(List("a", "b", "c"))
res0: List[(Int, String)] = List((1,a), (2,b), (3,c))
```  

##### partition  

 其作用是按照某个条件对集合进行切割，符合条件与不符合条件的都返回一个新集合  
 
```
scala> val numbers = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
scala> numbers.partition(_ %2 == 0)
res0: (List[Int], List[Int]) = (List(2, 4, 6, 8, 10),List(1, 3, 5, 7, 9))
```  

#####find 
 查找元素，但是，`它只返回第一个符合条件的元素`.

```
scala> numbers.find((i: Int) => i > 5)
res0: Option[Int] = Some(6)
```
 
##### drop 
 摘掉指定元素， 保留指定元素的后面部分:  

```
scala> numbers :+ 5
res18: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 5)

scala> numbers.drop(5)
res19: List[Int] = List(6, 7, 8, 9, 10, 11, 12, 13)
```  

##### dropWhile  
 删除第一个符合条件的元素，后面得元素自动忽略这个条件  
 
```
scala> numbers.dropWhile(_ % 2 != 0)
res0: List[Int] = List(2, 3, 4, 5, 6, 7, 8, 9, 10)
```  
我们要删除奇数，但是只有`1`被删除掉了，`3`被保留下来，因为它被`2`隔离了。  

##### foldLeft/foldRight

```
scala> numbers.foldLeft(0){(first:Int,second:Int)=> println("first="+first+",second="+second); first+second }
first=0,second=1
first=1,second=2
first=3,second=3
first=6,second=4
first=10,second=5
first=15,second=6
first=21,second=7
first=28,second=8
first=36,second=9
first=45,second=10
first=55,second=11
first=66,second=12
first=78,second=13
res40: Int = 91
```  

将集合中的元素累计相加， 比如， 左边是上一次两者之和，右边是接下来的元素值.. 

`foldRight` 则是与上面类似，只是顺序是从集合后面往前移动.

```
scala> numbers.foldRight(0){(first:Int,second:Int)=> println("first="+first+",second="+second); first+second }
first=13,second=0
first=12,second=13
first=11,second=25
first=10,second=36
first=9,second=46
first=8,second=55
first=7,second=63
first=6,second=70
first=5,second=76
first=4,second=81
first=3,second=85
first=2,second=88
first=1,second=90
res41: Int = 91
```

##### flatten 
 将多个集合进行合并  
 
```
scala> List(List(1,2,3),List(2,3,4,5)).flatten
res42: List[Int] = List(1, 2, 3, 2, 3, 4, 5)
```  

##### flatMap

将函数应用与每个元素 

```
scala> val nestedNumbers = List(List(1, 2), List(3, 4))
nestedNumbers: List[List[Int]] = List(List(1, 2), List(3, 4))

scala> nestedNumbers.flatMap(x => x.map(_ * 2))
res0: List[Int] = List(2, 4, 6, 8) 

scala> nestedNumbers.map((x: List[Int]) => x.map(_ * 2)).flatten
res1: List[Int] = List(2, 4, 6, 8)
```  

更多关于`flatMap`[Effective Scala#flatMap](http://twitter.github.com/effectivescala/#Functional programming-`flatMap`) 


### 扩展我们自己的集合操作 

```
def ourMap(numbers: List[Int], fn: Int => Int): List[Int] = {
  numbers.foldRight(List[Int]()) { (x: Int, xs: List[Int]) =>
    fn(x) :: xs
  }
}

scala> ourMap(numbers, timesTwo(_))
res0: List[Int] = List(2, 4, 6, 8, 10, 12, 14, 16, 18, 20)
```

其实就是定义一个我们自己的函数 .

##### 关于Map再多说两句 

 如果我们要从下面这个Map中过滤出，所有电话号码<200的集合，我们该怎么做？ 
 
```
scala> val extensions = Map("steve" -> 100, "bob" -> 101, "joe" -> 201)
extensions: scala.collection.immutable.Map[String,Int] = Map((steve,100), (bob,101), (joe,201))
```

方法一、  

```
scala> extensions.filter((namePhone: (String, Int)) => namePhone._2 < 200)
res0: scala.collection.immutable.Map[String,Int] = Map((steve,100), (bob,101))
```

利用`filter` 函数， 其中 `namePhone._2` 表示的是访问`元组`(filter中的类型其实返回的是元组，通过下标访问)中的value  

方法二、  
  利用模式匹配的力量，简洁易懂.

```
scala> extensions.filter({case (name, extension) => extension < 200})
res0: scala.collection.immutable.Map[String,Int] = Map((steve,100), (bob,101))
```


