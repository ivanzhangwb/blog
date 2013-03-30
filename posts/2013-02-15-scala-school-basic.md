---
layout: post
title: "Scala School--基础知识"
date: 2013-02-15 
permalink: "/2013/scala/scala-school-basic.html"
categories: 
- Scala
---
  
_练习题都来至于Twitter的[Scala-School](http://twitter.github.com/scala_school/index.html)_  

## 基础知识  
### 表达式  
```
scala> 1 + 1
res0: Int = 2
```  
res0 这个变量是由Scala的解释器自动创建的。 它的类型为Int, 值为 Integer 2. `几乎Scala中一切都是表达式`  

### 变量  
你也可以给表达式的结果赋予一个名字。 
```
scala> val two = 1 + 1
two: Int = 2
```  
`val`类型的变量，其值是不可变的 ， 可以使用`var`来表示可变的值.    

```
scala> var name = "steve"
name: java.lang.String = steve

scala> name = "marius"
name: java.lang.String = marius
```

### 函数  

可以使用`def`关键字来创建函数  

```
scala> def addOne(m: Int): Int = m + 1
addOne: (m: Int)Int
```  
在 Scala中，函数签名中，参数的类型需要你指定。   

```
scala> val three = addOne(2)
three: Int = 3
```  
对于没有参数的函数，你调用的时候，可以省略掉括号. 比如，下面的调用方式都是一样的。  

```
scala> def three() = 1 + 2
three: ()Int

scala> three()
res2: Int = 3

scala> three
res3: Int = 3
```  

###### 匿名函数  
```
scala> (x: Int) => x + 1
res2: (Int) => Int = <function1>
```  
Scala解释器中给匿名函数自动赋予了一个res2的名字， 你可以这样调用   

```
scala> res2(1)
res3: Int = 2
```  
你也可以把匿名函数赋予一个变量  

```
scala> val addOne = (x: Int) => x + 1
addOne: (Int) => Int = <function1>

scala> addOne(1)
res4: Int = 2 

```  
`addOne(1)` 表示调用addOne所表示的函数,把参数1传给它。  

若函数由多条语句来组成，你可以使用`{}`号来进行组合，如下:  

```
def timesTwo(i: Int): Int = {
  println("hello world")
  i * 2
}
```  
多语句的匿名函数也是一样的形式:   

```
scala> { i: Int =>
  println("hello world")
  i * 2
}
res0: (Int) => Int = <function1>
```

##### 函数的局部应用  
在Scala中，你可以使用`_`来对一个函数进行局部使用，比如:  

```
def adder(m: Int, n: Int) = m + n         //> adder: (m: Int, n: Int)Int
val add2 =adder(2,_:Int)                  //> add2  : Int => Int = function1>
add2(3)                                   //> res1: Int = 5
```
 可以看到，`add2`这变量，其实是把 `adder(2,_:Int)` 调用的结果保存起来了，但是`_`这个参数其实是一个占位符.  接下来， `add2(3)` 就相当于把 3 这个参数赋给了 `_` 了，其结果就相当于调用 `adder(2,3)`   
 
##### 柯里化函数  
 有时，我们需要给一个函数先指定一个参数，随后再指定另外一个参数的值。 就可以使用局部应用和柯里化函数来进行  
 
```
scala> def multiply(m: Int)(n: Int): Int = m * n
multiply: (m: Int)(n: Int)Int

scala> multiply(2)(3)
res0: Int = 6
```
 你可以直接指定2个参数来做，得到最终的结果，你也可以换种方式  
 
```
scala> val timesTwo = multiply(2) _
timesTwo: (Int) => Int = <function1>

scala> timesTwo(3)
res1: Int = 6
```  
可以将上面的例子，修改一下，得到:  

```
val t=(adder _).curried                   //> t  : Int => (Int => Int) = <function1>
t(2)(3)                                   //> res2: Int = 5
```

##### 可变参数  
有一个特殊的语法，可以使方法中同一类型的参数重复传入。  

```
def capitalizeAll(args: String*) = {
    args.map { arg =>
      arg.capitalize
    }
  }                                               //> capitalizeAll: (args: String*)Seq[String]
capitalizeAll("asdf","hello","take")            //> res3: Seq[String] = ArrayBuffer(Asdf, Hello, Take)
```


### Classes

```
scala> class Calculator {
     |   val brand: String = "HP"
     |   def add(m: Int, n: Int): Int = m + n
     | }
defined class Calculator

scala> val calc = new Calculator
calc: Calculator = Calculator@e75a11

scala> calc.add(1, 2)
res1: Int = 3

scala> calc.brand
res2: String = "HP"
``` 
这是一个简单类的例子，看上去非常简单. 
 
##### 构造函数  
构造函数的代码在方法定义之外， 如  

```
class Calculator(brand: String) {
  /**
   * A constructor.
   */
  val color: String = if (brand == "TI") {
    "blue"
  } else if (brand == "HP") {
    "black"
  } else {
    "white"
  }

  // An instance method.
  def add(m: Int, n: Int): Int = m + n
}
```  
你可以使用构造函数来对类进行初始化. 

```
scala> val calc = new Calculator("HP")
calc: Calculator = Calculator@1e64cc4d

scala> calc.color
res0: String = black
```

### 继承  
类的继承使用`extends`关键字  

```
class ScientificCalculator(brand: String) extends Calculator(brand) {
  def log(m: Double, base: Double) = math.log(m) / math.log(base)
}
```  
 重载方法
```
class EvenMoreScientificCalculator(brand: String) extends ScientificCalculator(brand) {
  def log(m: Int): Double = log(m, math.exp(1))
}
```

##### 抽象类  

```
scala> abstract class Shape {
     |   def getArea():Int    // subclass should define this
     | }
defined class Shape

scala> class Circle(r: Int) extends Shape {
     |   def getArea():Int = { r * r * 3 }
     | }
defined class Circle

scala> val s = new Shape
<console>:8: error: class Shape is abstract; cannot be instantiated
       val s = new Shape
               ^

scala> val c = new Circle(2)
c: Circle = Circle@65c0035b
```

### 特质
特质类似于Java的借口，用户与类进行混入

```
trait Car {
  val brand: String
}

trait Shiny {
  val shineRefraction: Int
}
``` 
 可以使用`extends`关键字来实现特质，而且可以使用`with`关键字来实现多个特质  
 
```
class BMW extends Car with Shiny {
  val brand = "BMW"
  val shineRefraction = 12
}
```
关于抽象类和特质的区别，可以在这里参考 ["stackoverflow:Scala traits vs abstract classes"](http://stackoverflow.com/questions/1991042/scala-traits-vs-abstract-classes)  

### 类型 
 可以使用范型方式来定义特质或者类
 
```
trait Cache[K, V] {
  def get(key: K): V
  def put(key: K, value: V)
  def delete(key: K)
}
```

方法也可以采用范型方式来定义

```
def remove[K](key: K)
```  

