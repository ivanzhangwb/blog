---
layout: post
date: 2012-12-16
layout: post
title: "BTrace原理之 -- VM Attach API"
categories:
- JVM
tags:
- JVM
---

 BTrace的特点之一就是可以动态Attach到一个运行的JVM进程上，然后根据BTrace脚本来对目标JVM进行相应的操作。  
  这篇文章主要关注  `JVM Attach API`  ，JVM的 Attach有两种方式：  
  
  - `指定javaagent参数`   
  - `运行时动态attach` 

#### 指定参数  
这种方式的特点就是在目标JVM启动时，就确定好了要加载什么样的代理对象。   
比如命令形式：`java -javaagent:xxxx.jar MainClass`  
用一个例子来说明下：

###### 一个普通程序入口  

```
public class TestMain {

    public static void main(String[] args) throws InterruptedException {
        System.out.println("Hello");
    }
}

```


##### Agent对象

```
import java.lang.instrument.Instrumentation;
import java.io.*;

public class TestAgent {
    public static void agentmain(String args, Instrumentation inst) 
    throws Exception {
        System.out.println("Args:" + args);
    }
    public static void premain(String args, Instrumentation inst) 
    throws    Exception {
    System.out.println("Pre Args:" + args);
    Class[] classes = inst.getAllLoadedClasses();
    for (Class clazz : classes) {
       System.out.println(clazz.getName());
    }
   }
}
```

这个类比较简单，最终它会在目标类的Main方法执行之前，执行premain方法，其主要动作是将以及加载的类答疑in出来。
我们需要将这个类打包成jar文件，以便在目标JVM启动时候，以参数形式指定给它。打成jar的同时，设定MANIFEST.MF文件的内容。 告知目标JVM该如何处理.  

>
Agent-Class: TestAgent  Premain-Class: TestAgent  Can-Redine-Classes: true  Can-Retransform-Classes: true
>

最终用jar命令将其打包。 比如 `jar cvmf MANIFEST.MF xxx.jar TestAgent` 

###### 使用方法: 
`java -javaagent:xxx.jar TestMain` 

####动态Attach，并load对应的Agent

这种方式与之前指定参数的不同在于，其可以在JVM已经运行的情况下，动态的附着上去，并可以动态加载agent。

同样我们也来看下一个例子：
 目标JVM运行的类, 这个类会隔一秒钟打印一句话。 JVM会一直运行（模拟已经运行的JVM进程）     

```
public class TestMain {	public static void main(String[] args) throws InterruptedException {          while(true){              Thread.sleep(10000);              new Thread(new WaitThread()).start();          }      }           static class WaitThread implements Runnable {      @Override      public void run() {           	System.out.println("Hello");     }          }  }
```

##### Agent对象  

``` 
import java.lang.instrument.Instrumentation;
import java.io.*;
public class TestAgent {
    public static void agentmain(String args, Instrumentation inst) 
    throws Exception {
        System.out.println("Args:" + args);
    }
    public static void premain(String args, Instrumentation inst) 
    throws Exception {
        System.out.println("Pre Args:" + args);
        Class[] classes = inst.getAllLoadedClasses();
        for (Class clazz : classes) {
            System.out.println(clazz.getName());
        }
    }
}
```


代码同上，不同的是，动态加载agent的情况下，被调用的是`agentmain`方法, 其会在JVMload的时候，被调用。 为了简单示例，只打印了传递给Agent的参数。

同样的，这个类也需要打包成jar。   

>Agent-Class: TestAgent  
Premain-Class: TestAgent  Can-Redine-Classes: true  Can-Retransform-Classes: true
>

##### 使用方式

 动态附着到对应的JVM需要使用到JDK的 [Attach API](http://docs.oracle.com/javase/6/docs/technotes/guides/attach/index.html)  

```
import com.sun.tools.attach.VirtualMachine;
public class Main {    public static void main(String[] args) throws Exception{  	VirtualMachine vm =null;  
	String agentjarpath = "/home/ivanzhangwb/test.jar"; //agentjar路径      vm = VirtualMachine.attach("9730");//目标JVM的进程ID（PID）      vm.loadAgent(agentjarpath, "This is Args to the Agent.");  
    vm.detach();    }  
}
```
 
最终我们一旦运行这个Main方法， 其就会动态的附着到我们对应的JVM进程中，并为目标JVM加载我们指定的Agent，以达到我们想做的事情，
比如BTrace就为在附着到目标JVM后，开启一个ServerSocket，以便达到与目标进程通讯的目的。

