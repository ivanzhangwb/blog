---
layout: post
date: 2012-12-07
layout: post
title: "理解CMS的GC Log"
permalink: "understand-cms-log"
categories:
- Java
tags:
- Java
---

>打印gc信息的参数: 
>` -XX:+PrintGCDetails ` 和 ` -XX:+PrintGCTimeStamps `      

## 正常的CMS GC周期

<img src="http://i01.lw.aliimg.com/3l/up/up3l_212a11e7_677_501.jpg" width="650px"/>
 
`39.910: [GC 39.910: [ParNew: 261760K->0K(261952K), 0.2314667 secs] 262017K->26386K(1048384K), 0.2318679 secs]`   
 新生代的GC执行。 新生代的占用量经过GC之后，从 `261760K` 下降到了 `0K` . 花费的时间 `0.2314667 `秒.  
  
`40.146: [GC [1 CMS-initial-mark: 26386K(786432K)] 26404K(1048384K), 0.0074495 secs]`  
 老年代的GC. CMS会在这里进行打上初始标记.会有一次StopWholeWorld的暂停. 内存占用量从`786432K` 下降到`26386K`。
  
`40.154: [CMS-concurrent-mark-start]`  
开始并发标记阶段。    

`40.683: [CMS-concurrent-mark: 0.521/0.529 secs]`  
Concurrent marking 花费 `0.521`秒  `0.529`秒中包含有`yeild`其他线程的时间  

`40.683: [CMS-concurrent-preclean-start]`  
开始PreClean 阶段. PreClean 也是并行发生的. 这个阶段进行的一些工作，可以为下一次的`stop-the-world`(remark阶段)减少一些工作  

`40.701: [CMS-concurrent-preclean: 0.017/0.018 secs]`
开始进行一些pre-clean的工作。 这行日志表示其耗费了 `0.017`秒  

`40.704: [GC40.704: [Rescan (parallel) , 0.1790103 secs]40.883: [weak refs processing, 0.0100966 secs] [1 CMS-remark: 26386K(786432K)] 52644K(1048384K), 0.1897792 secs]`  
`Stop-the-world`阶段，这个阶段重新扫描CMS堆里面剩余的对象。重新从root开始扫描并处理引用的对象。重新扫描花费了`0.1790103`秒,处理软引用耗费了`0.0100966`秒，而这整个阶段总归耗费了`0.1897792`秒.  

`40.894: [CMS-concurrent-sweep-start]`  
开始对以及消亡和未标记的对象进行sweeep。sweeping这个阶段与其他线程是并行运行的。  

`41.020: [CMS-concurrent-sweep: 0.126/0.126 secs]`  
sweep阶段耗费了`0.126`秒.  
  
`41.020: [CMS-concurrent-reset-start]`  
开始重置  

`41.147: [CMS-concurrent-reset: 0.127/0.127 secs]`  
CMS数据结构开始重新初始化，下次则会重新开始一个新的周期。这个阶段耗费了 `0.127`秒    

至此，一个正常的CMS周期运行结束了。

### CMS GC的其他情况

##### concurrent model failure
`197.976: [GC 197.976: [ParNew: 260872K->260872K(261952K), 0.0000688 secs]197.976: [CMS197.981: [CMS-concurrent-sweep: 0.516/0.531 secs]
(concurrent mode failure): 402978K->248977K(786432K), 2.3728734 secs] 663850K->248977K(1048384K), 2.3733725 secs]`  
这行GC日志显示的是一个请求失败的新生代GC处理。因为没有足够的空间来存储由新生代晋升上来的对象。这种现象称之为`full promotioin guarantee failure` 就此会在`197.981`处产生一次`FULL GC`，花费了 `2.3733725`秒 ，从而使CMS的空间从 `402978K->248977K`.  

`concurrent model failure`的避免办法：  
增在持久代大小或者在JVM启动时候使用`CMSInitiatingOccupancyFraction` 参数设置一个较小的值或者设置`UseCMSInitiatingOccupancyOnly`为true.  

`CMSInitiatingOccupancyFractioin`的值不能设置得比较低，较低的值会导致频繁的GC。  

有时出现`concurrent model failure`的原因是另外一种：“碎片”，老年代的内存是不连续的，从年轻代晋升到老年代需要一个连续的空间。CMS是一个非压缩(no-compacting)的的收集器。所以有可能会产生碎片。  


##### Trying a full collection because scavenge failed
Stop-the-world 也有可能在JNI临界释放的时候发生。 CMS可以在增量模式下运行来解决这个问题。  
开启`-XX:+CMSIncrementalModel` 这个选项可以让JVM运行在增量模式下面。 在这种模式下面，CMS收集器不会不占住处理器，会周期性暂停并发阶段，来将处理器让出来，以便application可以使用。

`2803.125: [GC 2803.125: [ParNew: 408832K->0K(409216K), 0.5371950 secs] 611130K->206985K(1048192K) icms_dc=4 , 0.5373720 secs]
2824.209: [GC 2824.209: [ParNew: 408832K->0K(409216K), 0.6755540 secs] 615806K->211897K(1048192K) icms_dc=4 , 0.6757740 secs]`  

上面的日志显示两次清理时间分别是537ms以及675ms.  

从JDK1.5开始后，有一个新的阶段 ` concurrent abortable preclean` 。 它会在 `concurrent preclean` 和 `remark` 之间，直到老年代达到我们期望的容量。  
比如如下的日志:  

```
7688.150: [CMS-concurrent-preclean-start]
7688.186: [CMS-concurrent-preclean: 0.034/0.035 secs]
7688.186: [CMS-concurrent-abortable-preclean-start]
7688.465: [GC 7688.465: [ParNew: 1040940K->1464K(1044544K), 0.0165840 secs] 1343593K->304365K(2093120K), 0.0167509 secs]
7690.093: [CMS-concurrent-abortable-preclean: 1.012/1.907 secs]
7690.095: [GC[YG occupancy: 522484 K (1044544 K)]7690.095: [Rescan (parallel) , 0.3665541 secs]7690.462: [weak refs processing, 0.0003850 secs] [1 CMS-remark: 302901K(1048576K)] 825385K(2093120K), 0.3670690 secs]
```  


参考链接 :  
<https://blogs.oracle.com/poonam/entry/understanding_cms_gc_logs>