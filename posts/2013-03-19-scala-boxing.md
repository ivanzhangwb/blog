---
layout: post
title: "Scala Boxing 问题"
permalink: "/2013/scala/scala-boxing.html"
date: 2013-03-19 
categories: 
- Scala
---

记录一下，自己碰到的Scala的Boxing疑问;
 
下午在测试某个功能的时候，发现有一个功能点报错：
 
 `--- Cause: java.sql.SQLException: Invalid parameter object type.  Expected 'com.laiwang.screen.dao.model.WallInviteMobile' but found 'scala.runtime.BoxedUnit'` 
  

看上去是类型错误了。打开源代码看到的是这样的。 原有代码是这样的: 
  
```
package com.laiwang.screen.dao.model

import scala.reflect.BeanProperty
import java.util.Date

class WallInviteMobile {
  @BeanProperty var id: String = _
  @BeanProperty var wallId: String = _
  @BeanProperty var mobile: String = _
  @BeanProperty var gmtCreate: Date = _
  @BeanProperty var gmtModified: Date = _
}

/**
 * 伴生对象
 */
object WallInviteMobile {
  def apply(id: String, wallId: String, mobile: String) {
    var wallinviteMobile = new WallInviteMobile
    wallinviteMobile.id = id
    wallinviteMobile.wallId = wallId
    wallinviteMobile.mobile = mobile
    wallinviteMobile
  }
}
```

很简单，一个类，为了方便和偷懒，我给了它一个伴生对象。

调用的地方，我是这么写的： 

```
 /**
   * 新增上墙邀请用户
   */
  def add(wallId: String, mobile: String) {
    var wallInviteMobile = WallInviteMobile(keyGenerator.getId(SEQUENCE_KEY), wallId, mobile)
    getSqlMapClientTemplate().insert("wall_invite_mobile.add", wallInviteMobile)
  }
```

 看来看去，好像没有什么问题啊。 难道我通过伴生对象产生的实例，Scala还会帮我包装一下？ 
 
 之后又在Scala REPL中尝试了如下的代码: 
 
```
scala> class WallInviteMobile {
     |  var id:String = _
     | }
defined class WallInviteMobile

scala> object WallInviteMobile {
     |   def apply() = {
     |   var ss = new WallInviteMobile()
     |   ss.id="1231"
     |   ss
     |  }
     | }
defined module WallInviteMobile
warning: previously defined class WallInviteMobile is not a companion to object WallInviteMobile.
Companions must be defined together; you may wish to use :paste mode for this.

scala> var sss=WallInviteMobile()
sss: WallInviteMobile = WallInviteMobile@77c5b2de
```
 
  结果，输出的是 `WallInviteMobile = WallInviteMobile@77c5b2de` ，这样更加让我疑惑了。 这里明明是我想要的类型，问题出来哪里呢？ 想了半天，没有结论 。 
  
  于是，我把我的代码放到了scala wroksheet 中，得到的效果是这样的：
  
![](http://i01.lw.aliimg.com/j4/yn/ynj4_22105bc5_917_369.880x760x75x2.jpg) 

  这下恍然大悟，之前也犯过类似的错误， 原来在定义伴生对象的apply方法的时候，没有加`=`号，导致通过伴生对象得到的是 `Unit`类型， 那么，既然是 `Unit`，那么为什么提示说，我得到的是 `BoxedUnit`类型呢? 
  

于是，我写了这样的代码来进行调试，想找到原因: 

```

import java.util.Date

class WallInviteMobile {
  var id : String = _
  var wallId : String = _
  var mobile : String = _
  var gmtCreate : Date = _
  var gmtModified : Date = _
}

object WallInviteMobile {
  def apply(id : String, wallId : String, mobile : String) {
    var wallinviteMobile = new WallInviteMobile
    wallinviteMobile.id = id
    wallinviteMobile.wallId = wallId
    wallinviteMobile.mobile = mobile
    wallinviteMobile
  }
}

object Test extends App {
  override def main(args : Array[String]) { 
    print(WallInviteMobile("112", "123", "132" )) //为了DEBUG.
  }
}
```
  
 查看了下 `Unit` 这个对象，其内部有两个比较重要的方法： 
 
```
object Unit extends AnyValCompanion {

  /** Transform a value type into a boxed reference type.
   *
   *  @param  x   the Unit to be boxed
   *  @return     a scala.runtime.BoxedUnit offering `x` as its underlying value.
   */
  def box(x: Unit): scala.runtime.BoxedUnit = scala.runtime.BoxedUnit.UNIT

  /** Transform a boxed type into a value type.  Note that this
   *  method is not typesafe: it accepts any Object, but will throw
   *  an exception if the argument is not a scala.runtime.BoxedUnit.
   *
   *  @param  x   the scala.runtime.BoxedUnit to be unboxed.
   *  @throws     ClassCastException  if the argument is not a scala.runtime.BoxedUnit
   *  @return     the Unit value ()
   */
  def unbox(x: java.lang.Object): Unit = ()

  /** The String representation of the scala.Unit companion object.
   */
  override def toString = "object scala.Unit"
}
```

BoxedUnit.UNIT 其就是一个BoxedUnit类型(这个貌似是一个Java类): 

```
public final class BoxedUnit implements java.io.Serializable {
    private static final long serialVersionUID = 8405543498931817370L;

    public final static BoxedUnit UNIT = new BoxedUnit();

    public final static Class<Void> TYPE = java.lang.Void.TYPE;

    private BoxedUnit() { }

    public boolean equals(java.lang.Object other) {
	return this == other;
    }

    public int hashCode() {
	return 0;
    }

    public String toString() {
	return "()";
    }
}

```

那么，我的问题是： 
 
`Scala在什么时候调用了Unit.box方法？`

在提问之前，我自己曾经想过，是否是在编译的时候，Scala编译器默认将`Unit`类型编译成了 `BoxedUnit`类型， 于是我编译了下我的代码: 

```
exercise - master! $> javap WallInviteMobile$                                                                                           
Compiled from "WallInviteMobile.scala"
public final class exercise.WallInviteMobile$ extends java.lang.Object implements scala.ScalaObject{
    public static final exercise.WallInviteMobile$ MODULE$;
    public static {};
    public void apply(java.lang.String, java.lang.String, java.lang.String);
}
```
看到的apply方法实际是返回的 `void` 类型, 于是我就困惑了。 求解

* Scala在什么时候调用了Unit.box方法？
* BoxedUnit与Java中void 的对应关系?  

------

下面是同事的回帖:  
(1)
 例子：  
 
```
 scala> 
def f() { 
  val a=2; 
  println(a.getClass);  
  def f2(any:Any){println(any.getClass)}; //定义嵌套函数
  f2(a); 
  println(a.getClass) 
}
f: ()Unit

scala> f
int
class java.lang.Integer
int

scala> 
def f() { 
  val a=(); 
  println(a.getClass); 
  def f2(any:Any){println(any.getClass)}; 
  f2(a); 
  println(a.getClass) 
}
f: ()Unit

scala> f
void
class scala.runtime.BoxedUnit
void
```


因为 int 和 Unit 都是 AnyVal 类型，它们都是可以被Boxing/UnBoxing 的，
但是测试一下，发现AnyVal 类型的数据在传入任何函数时，都会引起Boxing
即使指明参数类型就是一个Any 或者 AnyVal 都会发生，再比如：

```
scala> def f3(a:AnyVal) { println(a.getClass) }
f3: (a: AnyVal)Unit

scala> f3(2)
class java.lang.Integer //还是发生了Boxed
```

除非在声明参数的时候就是具体的Int, Unit 类型，才不会发生Boxing

```
scala> def foo(a:Unit) { println(a.getClass) }
foo: (a: Unit)Unit

scala> foo(())
void

scala> def foo(a:Int) { println(a.getClass) }
foo: (a: Int)Unit

scala> foo(2)
int
```


至于为什么Int明明符合Any/AnyVal 类型，却在运行时仍会Boxing，这个就不知道scala的runtime是处于什么考虑了。
我原本还因为参数指定为Any，传入的Int类型并不会立即Boxing，除非后边有逻辑需要boxing，但现在看执行函数时runtime是立即就做了boxing了。


(2)
分析一下为什么 Any类型的变量在被赋值一个Int值时会发生boxing，归根到底，是因为Any/AnyVal 类型被翻译为java时是使用的 Object 这个引用类型

```
scala -Xprint:jvm -e 'val a:Any=100'
...
  final class Main$$anon$1 extends Object {
    private[this] val a: Object = _; //Any/AnyVal都被转为Object 
    <stable> <accessor> private def a(): Object = Main$$anon$1.this.a;
    def <init>(): anonymous class anon$1 = {
      Main$$anon$1.super.<init>();
      Main$$anon$1.this.a = scala.Int.box(100); //需要boxing
      ()
    }
  }
```


而 Int，Float， Unit这些类型被翻译为java代码时，就是对应的java基本类型的int, float, void

```
scala -Xprint:jvm -e 'val b:Int=100'  
...
final class Main$$anon$1 extends Object {
    private[this] val b: Int = _;   // 就是 java里的int类型，
    <stable> <accessor> private def b(): Int = Main$$anon$1.this.b;
    def <init>(): anonymous class anon$1 = {
      Main$$anon$1.super.<init>();
      Main$$anon$1.this.b = 100; //不需要boxing
      ()
    }
  }
```

所以在函数调用时，参数的传递是值拷贝，这个过程触发了boxing 。