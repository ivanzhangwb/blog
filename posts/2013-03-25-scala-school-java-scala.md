---
layout: post
title: "Scala School Java与Scala互操作"
permalink: "/2013/scala/scala-school-with-java.html"
date: 2013-03-25 
categories: 
- Scala
---

## Javap
  `javap`是JDK的一个反编译工具，通常可以将类文件进行反编译，以便直观的查看到类中的定义，使用起来也非常的简单: 
  
  
```
scala $> javap MyTrait                                                                                        [ ruby-1.9.3-p327 ]
Compiled from "MyTrait.scala"
public interface MyTrait extends scala.ScalaObject{
    public abstract void MyTrait$_setter_$upperTraitName_$eq(java.lang.String);
    public abstract java.lang.String traitName();
    public abstract java.lang.String upperTraitName();
}
```

也快要查看类对应的字节码:  

```
scala $> javap -c MyTrait\$class                                                                              [ ruby-1.9.3-p327 ]
Compiled from "MyTrait.scala"
public abstract class MyTrait$class extends java.lang.Object{
public static void $init$(MyTrait);
  Code:
   0:	aload_0
   1:	aload_0
   2:	invokeinterface	#12,  1; //InterfaceMethod MyTrait.traitName:()Ljava/lang/String;
   7:	invokevirtual	#17; //Method java/lang/String.toUpperCase:()Ljava/lang/String;
   10:	invokeinterface	#21,  2; //InterfaceMethod MyTrait.MyTrait$_setter_$upperTraitName_$eq:(Ljava/lang/String;)V
   15:	return

}
```  

 一旦在java端有什么问题，你就可以尝试用`javap`来一探究竟。  
 
## Classes

 在java端使用Scala的类有四个值的注意的方面:  
 
*  Class parameters
*  Class vals
*  Class vars
*  Exceptions

我们使用一个简单的Scala类来做下实例:  

```
 import java.io.IOException
 import scala.throws
 import scala.reflect.{BeanProperty,BooleanBeanProperty}
 class SimpleClass(name:String,val acc:String,@BeanProperty var mutable:String){
  val foo="foo"
  var bar="bar"
  @BeanProperty
  val fooBean = "foobean"
  @BeanProperty
  var barBean = "barbean"
  @BeanProperty
  var awesome =true
  def dangerFoo() ={
     throw new IOException("SURPRISE!")
  }
  @throws(classOf[IOException])
  def dangerBar() = {
    throw new IOException("NO SURPRISE!")
  }
 }
```

### Class parameters
  通常来说，类的构造参数只能在类的内部访问到. 将类参数定义为 `val/var`的形式，与下面的代码是同样的意义. 
  
```
class SimpleClass(acc_: String) {
  val acc = acc_
}
```

### Var 和 Val

 如果你定义变量为 `val` ，则你可以在 `java` 方面使用如下的方式来调用. 比如:  

`val foo` 则你可以使用 `foo()` 方法来调用.

 如果你定义变量为 `var` , 则你可以在 `java` 方面使用 `$_eq` 结尾的方式来调用, 
`var foo` 则可以使用  `foo$_eq("xxx")` 即可.


## BeanProperty 
 
 你可以使用`@BeanProperty` 这个注解来给变量自动生成`get/set`方法， 对于 `boolean` 类型的，你可以使用 `@BooleanBeanProperty` 注解来生成.
 
## Exceptions
  Scala中没有受检查的异常,但是Java中有。不过，当你想在Java端捕获一个异常，你仍然可以按照Java的方式来做. 
  
```
// exception erasure!
try {
   s.dangerFoo();
} catch (IOException e) {
   // UGLY
}
```

不过在编译阶段，由于方法体没有声明抛出异常，通常在编译阶段是无法得知其会抛出异常，很可能导致忽略`try`, 那么更好的方法是使用Scala当中提供的`@throws`注解来帮助编译器识别。 比如 `SimpleClass中的dangerBar方法`。

PS： 支持与Java互操作的注解列表在 [Scala与Java交互注解](http://www.scala-lang.org/node/106)有更多的描述.


## Traits
  
```
trait MyTrait {
  def traitName:String
  def upperTraitName = traitName.toUpperCase
}
```
 这是一个Scala的特质， 其中有一个抽象方法`traitName` ，和一个已经实现的方法`upperTraitName` ，Scala会为我们生成什么呢？ 
 
* 一个名称为MyTrait的接口
* 一个名称为MyTrait$class的实现类

反编译后，接口的定义如下: 

```
scala $> javap MyTrait                                                                                       
Compiled from "MyTrait.scala"
public interface MyTrait extends scala.ScalaObject{
    public abstract java.lang.String traitName();
    public abstract java.lang.String upperTraitName();
}
```

那么实现类`MyTrait$class`的反编译结果如下： 

```
scala $> javap MyTrait\$class                                                                                
Compiled from "MyTrait.scala"
public abstract class MyTrait$class extends java.lang.Object{
    public static java.lang.String upperTraitName(MyTrait);
    public static void $init$(MyTrait);
}
```

实现类当中，有一个需要传入`MyTrait`接口的静态的方法。 于是，如果我们需要在JAVA中实现`MyTrait`特质的话，我们就需要做两件事情: 

* 实现MyTrait
* 实现upperTraitName方法，（若是需要默认的实现，则可以代理下`MyTrait$class`类）

```
public class JTraitImpl implements MyTrait {
    private String name = null;

    public JTraitImpl(String name) {
        this.name = name;
    }

    public String traitName() {
        return name;
    }
    
    public String upperTraitName() {
        return MyTrait$class.upperTraitName(this);
    }
}
```

## Objects

  对象在Scala中是用来表示单例与静态域的方式. 一个Scala的对象，在编译之后，会带有一个`$`的后缀.
  
  
```
class TraitImpl(name: String) extends MyTrait {
  def traitName = name
}

object TraitImpl {
  def apply = new TraitImpl("foo")
  def apply(name: String) = new TraitImpl(name)
}
```

则，我们在Java中可以这样来进行访问: 

```
MyTrait foo = TraitImpl$.MODULE$.apply("foo");
```  

我们将类进行反编译后，得到的输出是这样的 

```
scala $> javap TraitImpl\$                                                                                   
Compiled from "TraitImpl.scala"
public final class TraitImpl$ extends java.lang.Object implements scala.ScalaObject{
    public static final TraitImpl$ MODULE$;
    public static {};
    public TraitImpl apply();
    public TraitImpl apply(java.lang.String);
}
```

 可以看到其内部没有静态方法，取而代之的是一个名为`MOUDULE$`的静态的字段， 其内部其实就是针对这个静态的类型做了一个代理，将你真实的请求代理给这个静态类型去处理.
 
 
## 闭包函数  
  Scala中一个比较重要的特性是将函数作为头等公民来看待的。
  
```
class ClosureClass {
  def printResult[T](f: => T) = {
    println(f)
  }

  def printResult[T](f: String => T) = {
    println(f("HI THERE"))
  }
}
```  

在Scala中，我们可以这个来调用它: 

```
val cc = new ClosureClass
cc.printResult{"HI MOM"}
```

我们将类进行反编译后，得到的信息如下: 

```
Compiled from "ClosureClass.scala"
public class ClosureClass extends java.lang.Object implements scala.ScalaObject{
    public void printResult(scala.Function0);
    public void printResult(scala.Function1);
    public ClosureClass();
}
```

编译后，我们的知 , `f: => T`被转换成了`Function0` 而 `f:String => T`被转换成了`Function1` . Scala中，预定义了 `Function0~Function22`支持从0~22个参数的方法。 

现在，我们来看看，在Java中，如何来对这个类的方法进行调用.

```
    @Test public void closureTest() {
        ClosureClass c = new ClosureClass();
        c.printResult(new AbstractFunction0() {
                public String apply() {
                    return "foo";
                }
            });
        c.printResult(new AbstractFunction1<String, String>() {
                public String apply(String arg) {
                    return arg + "foo";
                }
            });
    }

```	

结论很有趣， 结果Scala提供了`AbstractFunction0 和 AbstractFunction1` 以便我们可以传递给Scala方法.


 








