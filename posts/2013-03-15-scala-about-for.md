---
layout: post
title: "Scala中的for表达式"
permalink: "/2013/scala/scala-school-about-for.html"
date: 2013-03-15 
categories: 
- Scala
---

在Scala中`for`语句可以用来做列表和集合的循环，其作用与Java类似，但是，除了这些基本功能以外，Scala中还对`for`语法进行了增强，可以在遍历的同时进行筛选、查询等等操作。

## for循环 

最常用的方式是利用`for`语法来进行遍历列表，比如这样: 

```
scala> val list =List(1,2,3,4,5)
list: List[Int] = List(1, 2, 3, 4, 5)

scala> for(i<-list) println(i)
1
2
3
4

```

它可以简单的遍历集合类型

## for的守卫

除了上述的基本用法以外， `for`语法中还提供了一种称之为守卫的方式， 在遍历的同时，可以给定一些条件，以便进行筛选， 在语法上，利用`yield`关键字来进行操作.

还是上面的例子，我们在遍历的时候，进行一下筛选: 

```
scala> val newlist = for(i<-list;if i >2) yield i
newlist: List[Int] = List(3, 4, 5)
```
 
可以看到，通过`yield`的筛选后，得到了我们需要的一个新的列表。 
当然，我们还可以进行其余的操作，比如，我需要得到一个包含有元组的新的列表

```
scala> val newlist = for(i<-list;if i >2) yield (i+"-hi" -> i)
newlist: List[(String, Int)] = List((3-hi,3), (4-hi,4), (5-hi,5))
```

## 使用for表达式做查询

结合yield的使用，我们来看一个利用 `for yield`来进行查询的例子 

```
scala> case class Book(title:String,author:String*)
defined class Book

scala> val books:List[Book] =  List(
     |  Book(
     |    "Structure and Interpretation of Computer Programs",
     |    "Abelson ,Harold","Sumsman"
     |  ),
     |  Book(
     |    "BookB",
     |    "AuthorB","AuthorB2"
     |  ),
     |  Book(
     |    "BookC",
     |    "AuthorC","Author3"
     |  )
     | )
books: List[Book] = List(Book(Structure and Interpretation of Computer Programs,WrappedArray(Abelson ,Harold, Sumsman)), Book(BookB,WrappedArray(AuthorB, AuthorB2)), Book(BookC,WrappedArray(AuthorC, Author3)))

scala> for(b<-books; a<- b.author
     |   if a startsWith "Author" )
     |  yield b.title
res0: List[String] = List(BookB, BookB, BookC, BookC)

scala> for(b1<-books;b2<-books if b1 != b2;
     | a1<-b1.author ; a2<-b2.author if a1 == a2)
     | yield a1
res1: List[String] = List()
```

可以看到，我们遍历对象列表，获取我们需要的对象, 第一个`for yield` 是为了查询出符合某个名称的书籍。
第二个查询是为了找到写过2本以上书的作者。   


## for的转译

在Scala内部，针对for表达式，其实Scala编译器在背后做了很多事情，比如来说： 
针对for ,  

* 带有 `yield`关键字的for表达式，在背后会被编译器转译为 `map,flatMap,filter`等高阶函数的调用
* 没有带`yield`关键字的则会被编译成 `filter与foreach`的调用.  

#### for的情况 

那我们来查看一下 之前的例子中，单纯的for表达式会被转译成什么样子：
源代码 如下： 

`val list=List(1,2,3) ; for(i<-list) print(i)` 

翻译后的代码为：  

```                              
 [[syntax trees at end of typer]]// Scala source: scalacmd6746774744676454543.scala
package <empty> {
  final object Main extends java.lang.Object with ScalaObject {
    def this(): object Main = {
      Main.super.this();
      ()
    };
    def main(argv: Array[String]): Unit = {
      val args: Array[String] = argv;
      {
        final class $anon extends scala.AnyRef {
          def this(): anonymous class $anon = {
            $anon.super.this();
            ()
          };
          private[this] val list: List[Int] = immutable.this.List.apply[Int](1, 2, 3);
          private <stable> <accessor> def list: List[Int] = $anon.this.list;
          $anon.this.list.foreach[Unit](((i: Int) => scala.this.Predef.print(i)))
        };
        {
          new $anon();
          ()
        }
      }
    }
  }
}
```



可以看到，for表达式被编译成了 `this.list.foreach`的调用. 

##### for yield的情况
再来看看 `for yield`的情况 :  

`val list=List(1,2,3) ; for(i<-list) yield i`

编译后的代码:  

```
[[syntax trees at end of typer]]// Scala source: scalacmd8186341042125019106.scala
package <empty> {
  final object Main extends java.lang.Object with ScalaObject {
    def this(): object Main = {
      Main.super.this();
      ()
    };
    def main(argv: Array[String]): Unit = {
      val args: Array[String] = argv;
      {
        final class $anon extends scala.AnyRef {
          def this(): anonymous class $anon = {
            $anon.super.this();
            ()
          };
          private[this] val list: List[Int] = immutable.this.List.apply[Int](1, 2, 3);
          private <stable> <accessor> def list: List[Int] = $anon.this.list;
          $anon.this.list.map[Int, List[Int]](((i: Int) => i))(immutable.this.List.canBuildFrom[Int])
        };
        {
          new $anon();
          ()
        }
      }
    }
  }
}
```
可以看到，带`yield`的被编译成了对`this.list.map`方法的调用 .  






