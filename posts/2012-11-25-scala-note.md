---
date: 2012-11-25
layout: post
title: "Scala笔记"
permalink: "/2012/scala/scala-note.html"
categories: 
- Scala
---


#### Scala Notes
1. Scala是纯粹格式的面向对象语言:每个值都是对象，每个操作都是方法调用。 for example:  
`1+2` : 实际调用了定义再Int类里面一个名为 + 的方法 .  

2. `imperative` 表示数据为不可变的。  


3. Scala里面，分号是可选的，可写也可不写(通常不写)  


4. Scala中，定义一个类不需要像Java那么繁琐，比如你可以这样定义一个类:  
`class MyClass(index:Int,name:String)` 它会为`MyClass` 自动生成两个私有字段.  

5. Scala解释器的过程为:`读取--求值--打印--循环`(REPL). 而Scala解释器实际上是将输入的内容快速编译成字节码，然后将这段字节码交给JVM执行.  


6. 使用`val` 定义的变量，是一个`常量`， 而申明可变变量，可以使用 `var` . Scala中鼓励使用`val`，除非真的需要改变它的内容.  


7. 在Scala中， 变量或者函数的类型，都建议存放到变量的后面，比如`greeting:String`,而Java中则是相反的 `String greeting`.  


8. Scala中也含有`Boolean、Char、Short、Int、Long、Float、Double`和`Boolean` 但是与Java所不同的是，这些全部都是`类`,因为在Scala中，它不刻意区分 基本类型 和 引用类型. 在Scala中，不需要包装类型，基本类型到引用类型的转换是Scala编译器的工作. 例如： 你创建一个Int的数组，最终到虚拟机中，得到的是一个int[]数组.  


9. Scala底层使用 `java.lang.String` 来字符串，不过它通过`StringOps`类给字符串加上了上百种操作. 比如 `"Hello".intersect("World")` 其中 `"Hello"`会被隐式的转换成一个`StringOps`对象,接着`StringOps`类的`intersect`方法被应用. 类似的类还有比如`RicInt,RichDouble,RichChar`等等.  


10. 关于操作符 ， Scala中没有`++/ --` 取而代之的是 `+=1 或 -=1` , 另外，比如说 `1+2`，其实它是表示 调用了 `1的一个名称为 + 的方法`，Scala中方法的名称没有限制， 你可以这样来调用方法,` a 方法 b` 或 `a.方法(b)` ，类似的, `a+b ==> a.(b)` ,由于操作符即只是表示一个方法名字，所以在Scala中，我们可以为操作符进行重载.  


11. 除了方法之外，Scala还提供函数，`sqrt(2)`等等类似的， 它并不隶属于某一个类，所以不用加上前缀. 类似的数学函数是需要引入包的 `import scala.match._ ` “_”表示通配符. Scala
中没有静态方法,不过它有一个特性，叫做`单例对象`

12. `apply`方法, 简单来说，就是 `s(i)` 表示取s这个字符串的第i个字符。 比如`"Hello"(4)` 它的内部其实是使用了 `"Hello".apply(4)` 这个方法来实现的.

13. Scala中`if/else`表达式是有值，这个值就是跟在if或else之后的表达式值。比如说: `val s = if(x>0) 1 else -1` . Scala中每个表达式都有一个类型。 举例来说: `if (x > 0) "positive" else -1` 其返回的类型是公共超类型 `Any` .

14. 在Scala中，每个表达式都应该有某种值。 其解决方案就是引入了一个`Unit`类，写作 `()` . 比如不带else的if 语句 : `if (x > 0 ) 1   ==> if (x > 0) 1 else ()` 

15. `()` 可以把它当成 Java 里面的`void` 关键字， 其含义可以理解为 `无有用值` 

16. `{}` 块表达式 ，其结果也是一个表达式。 块中最后一个表达式的值就是块的值。 

17. `readLine` 从控制台读取一行输入。 类似的还有诸如 `readInt\readDouble\readByte\readShort\readLong` 等等函数。 `readLine`可以带有一个参数作为提示信息，比如 `readLine("Your Name:")` 

18. Scala中对于for循环的使用语法如下：`for( i <-1 to n){ r = r * i}` `for( i <- 表达式)` 让变量i遍历 <- 右边的表达式所有值. 还有类似的`util` 关键字 : `for (i <- 0 util s.length)` 表示i会循环到 `s.length-1` 这个最后的元素.

19. 循环推导: 如果for循环的循环体以yield开始，则该循环会构造出一个集合，每次迭代生成集合的一个值 `for(i <- 1 to 10) yield i % 3` 会构造出集合 `Vector(1,2,0,1,2,0,1,2,0,1)`  这类循环则称为for推导式.

20. Scala中除了方法外，还有函数, Java中函数的概念只能通过静态方法来实现。 Scala中定义函数需要 : 函数名、参数和函数体。 `def abs(x:Double) = if (x>=0) x else -x` 如果函数需要多个表达式来完成，则表达式的最后一句默认为函数的返回值。 `注意： Scala中，在函数最后加上 return 并不常见`

21. Scala对于不返回值的函数有特殊的表示法 . 如果函数体包含在花括号当中，但没有前面的=号，那么返回的就是Unit。 这样的函数称之为`过程`. 过程不返回值。

22. 懒值: 关键字 `lazy` 当你将变量声明为懒值后，它的初始化将被推迟，直到我们首次对它取值。 `val words = scala.io.Source.fromFile("/usr/share/dict/words").mkString`在words被定义时即取值 ; `lazy val words ….` 在worlds被首次使用时取值;  `def words xxxx` 在每一次words被使用的时候取值.

23. `val nums = new Array[Int](10)` 10个整数的数组,所有元素初始化为0. `val a = new Array[String](10)` 10个元素的字符串数组,所有元素初始化为null. `val s = Array("Hello","World")` 长度为2的Array,类型是推导出来的. `s(0)="GoodBye"` 这样之后,数组的元素则变成了 `val s = Array("GoodBye","World")` 了.

24. 可变数组: `ArrayBuffer`.  

```
	import scala.collection.mutable.ArrayBuffer
	val b = ArrayBuffer[Int]()
	b+=1  // ArrayBuffer(1)
	b+= (1,2,3,5)
	b++= Array(8,13,21)
``` 