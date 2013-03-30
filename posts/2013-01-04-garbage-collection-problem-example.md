---
layout: post
date: 2013-01-04
layout: post
title: "诊断GC问题(1)"
permalink: "garbage-collection-problem"
categories:
- JVM
tags:
- JVM
---

分析问题之前，我们需要首先了解GC日志的具体格式.  
在命令行下对JVM使用`-verbosegc -XX:+ PrintGCDetails`参数可以打开GC的具体输出.而具体的GC日志输出格式如下：   

<img src="http://i01.lw.aliimg.com/pt/vy/vypt_0e33ec22_973_101.jpg" width="650px"/>

  
## 新生代过小引发的问题
 有如下日志： 
 
```
[GC [DefNew: 4032K->64K(4032K), 0.0429742 secs] 9350K->7748K(32704K), 0.0431096 secs]
[GC [DefNew: 4032K->64K(4032K), 0.0403446 secs] 11716K->10121K(32704K), 0.0404867 secs]
[GC [DefNew: 4032K->64K(4032K), 0.0443969 secs] 14089K->12562K(32704K), 0.0445251 secs]
```

我们可以从中知道，整个堆的大小为32M， 新生代大小为4M.  
以第一行为例:

```
[GC [DefNew: 4032K->64K(4032K), 0.0429742 secs] 9350K->7748K(32704K), 0.0431096 secs] 
```  
我们可以从中看到 , 年轻带回收了 `3968k` 内存， 而整个堆则回收了 `1602k`  

为什么整个堆回收的内存会比新生代小呢？  
原因就在，有些对象虽然从新生代移走了，但是并没有被GC掉，而是晋升到了老年代中.
所以，这行日志实际显示， 每次YGC，都只回收了 `1602k`,也就是`1602/4032=0.4` 实际在新生代中，只有40%的对象是可以GC的. 其余都晋升到老年代或者S区去了.

我们增大一下新生代到8M试试：  

```
[GC [DefNew: 8128K->64K(8128K), 0.0453670 secs] 13000K->7427K(32704K), 0.0454906 secs]
[GC [DefNew: 8128K->64K(8128K), 0.0388632 secs] 15491K->9663K(32704K), 0.0390013 secs]
[GC [DefNew: 8128K->64K(8128K), 0.0388610 secs] 17727K->11829K(32704K), 0.0389919 secs]
```  

从这个日志里面看， 每次YGC之后，整个堆被回收掉的对象有: `13000-7427=5573k`, 它占新生代的 `68%`   

所以我们可以从上面的日志可以了解到， 有效的增加新生代的大小，在YGC之后有利于空余内存空间的增长。

#### 对应用暂停时间的影响
` -XX:+PrintGCApplicationConcurrentTime and -XX:+ PrintGCApplicationStoppedTime`    
这两个参数可以将GC时应用运行时间与GC时应用暂停时间分别打印出来  

比如如下日志 ，新生代为4M时：

```
Application time: 0.5291524 seconds
[GC [DefNew: 3968K->64K(4032K), 0.0460948 secs] 7451K->6186K(32704K), 0.0462350 secs]
Total time for which application threads were stopped: 0.0468229 seconds
Application time: 0.5279058 seconds
[GC [DefNew: 4032K->64K(4032K), 0.0447854 secs] 10154K->8648K(32704K), 0.0449156 secs]
Total time for which application threads were stopped: 0.0453124 seconds
Application time: 0.9063706 seconds
[GC [DefNew: 4032K->64K(4032K), 0.0464574 secs] 12616K->11187K(32704K), 0.0465921 secs]
Total time for which application threads were stopped: 0.0470484 seconds
```  

而8M的新生代时:  

```
Application time: 1.3874623 seconds
[GC [DefNew: 8064K->63K(8128K), 0.0509215 secs] 11106K->5994K(32704K), 0.0510972 secs]
Total time for which application threads were stopped: 0.0517092 seconds
Application time: 1.5225065 seconds
[GC [DefNew: 8127K->63K(8128K), 0.0432982 secs] 14058K->8273K(32704K), 0.0434172 secs]
Total time for which application threads were stopped: 0.0440447 seconds
Application time: 1.4263524 seconds
[GC [DefNew: 8127K->64K(8128K), 0.0363538 secs] 16337K->10381K(32704K), 0.0364811 secs]
Total time for which application threads were stopped: 0.0369103 seconds
```
可以从上面的日志对比得到， 8M大小的新生代，应用的运行时间明显要长一些。


## 新生代过大引发的问题  

当我们把新生代增加到 16M 的时候，会有什么事情发生？  

```
[GC [DefNew: 16000K->16000K(16192K), 0.0000574 secs][Tenured: 2973K->2704K(16384K), 0.1012650 secs] 18973K->2704K(32576K), 0.1015066 secs]
[GC [DefNew: 16000K->16000K(16192K), 0.0000518 secs][Tenured: 2704K->2535K(16384K), 0.0931034 secs] 18704K->2535K(32576K), 0.0933519 secs]
[GC [DefNew: 16000K->16000K(16192K), 0.0000498 secs][Tenured: 2535K->2319K(16384K), 0.0860925 secs] 18535K->2319K(32576K), 0.0863350 secs]
```  

从日志里面，我们可以看到 ，新生代的gc没有回收掉内存,`16000k->16000k`,而老年代(Tenured)的回收情况也不乐观 `2973k->2704k` ,所以这个例子里面，由于老年代占用的内存已经比较多了, 触发了一次full gc(major gc),最终堆回收的内存为`18973k->2704k` 

### 老年代设置大一点还是小一点?  

当堆大小为32M，老年代为24M时（新生代为8M），下面的日志显示，应用的暂停时间为0.13秒 

 
```
[GC [DefNew: 8128K->8128K(8128K), 0.0000558 secs][Tenured: 17746K->2309K(24576K), 0.1247669 secs] 25874K->2309K(32704K), 0.1250098 secs]
```  

当堆大小为64M，老年代为56M时(新生代为8M),应用的暂停时间增加到了0.21秒  

```
[GC [DefNew: 8128K->8128K(8128K), 0.0000369 secs][Tenured: 50059K->5338K(57344K), 0.2218912 secs]
58187K->5338K(65472K), 0.2221317 secs]
```  

很明显，这是因为堆越大，需要回收的对象就越多，暂停时间也就会相应的延长.

但是， 当堆分别为32M和64M时，它们的触发频率也不一样了。 
`-XX:+ PrintGCTimeStamps` 带上这个参数，可以看到每次GC出发的时间戳.  

如下， 32M的堆大小，full gc(major GC)出发的频率在 10~11秒左右  

```
111.042: [GC 111.042: [DefNew: 8128K->8128K(8128K), 0.0000505 secs]111.042: [Tenured: 18154K->2311K(24576K), 0.1290354 secs] 26282K->2311K(32704K), 0.1293306 secs]
122.463: [GC 122.463: [DefNew: 8128K->8128K(8128K), 0.0000560 secs]122.463: [Tenured: 18630K->2366K(24576K), 0.1322560 secs] 26758K->2366K(32704K), 0.1325284 secs]
133.896: [GC 133.897: [DefNew: 8128K->8128K(8128K), 0.0000443 secs]133.897: [Tenured: 18240K->2573K(24576K), 0.1340199 secs] 26368K->2573K(32704K), 0.1343218 secs]
144.112: [GC 144.112: [DefNew: 8128K->8128K(8128K), 0.0000544 secs]144.112: [Tenured: 16564K->2304K(24576K), 0.1246831 secs] 24692K->2304K(32704K), 0.1249602 secs]
```  

当堆大小为64M时，full gc(major gc)的触发频率在 30秒左右:  

```
90.597: [GC 90.597: [DefNew: 8128K->8128K(8128K), 0.0000542 secs]90.597: [Tenured: 49841K->5141K(57344K), 0.2129882 secs] 57969K->5141K(65472K), 0.2133274 secs]
120.899: [GC 120.899: [DefNew: 8128K->8128K(8128K), 0.0000550 secs]120.899: [Tenured: 50384K->2430K(57344K), 0.2216590 secs] 58512K->2430K(65472K), 0.2219384 secs]
153.968: [GC 153.968: [DefNew: 8128K->8128K(8128K), 0.0000511 secs]153.968: [Tenured: 51164K->2309K(57344K), 0.2193906 secs] 59292K->2309K(65472K), 0.2196372 secs]
```

所以，堆大小的设置需要有所取舍。 小的堆暂停时间较短，而大的堆则吞吐率会更好一些。


### 合适的GC策略选择器

一旦你发现YGC的暂停时间比较长的话，建议使用`-XX:+UseParNewGC` 的GC策略.
其是并发来进行minor gc(young gc)的。比如如下的日志:

```
497.905: [GC 497.905: [ ParNew: 64576K->960K(64576K), 0.0255372 secs] 155310K->93003K(261184K), 0.0256767 secs]
506.305: [GC 506.305: [ParNew: 64576K->960K(64576K), 0.0276291 secs] 156619K->94267K(261184K), 0.0277958 secs]
514.565: [GC 514.565: [ParNew: 64576K->960K(64576K), 0.0261376 secs] 157883K->95711K(261184K), 0.0262927 secs]
522.838: [GC 522.838: [ParNew: 64576K->960K(64576K), 0.0316625 secs] 159327K->97331K(261184K), 0.0318099 secs]
```  

如果 major gc的暂停时间较长，建议使用`-XX:+UseConcMarkSweepGC` CMS的GC策略.

对应的日志可能如下:

```
[GC [ParNew: 64576K->960K(64576K), 0.0377639 secs] 140122K->78078K(261184K), 0.0379598 secs]
[GC [ParNew: 64576K->960K(64576K), 0.0329313 secs] 141694K->79533K(261184K), 0.0331324 secs]
[GC [ParNew: 64576K->960K(64576K), 0.0413880 secs] 143149K->81128K(261184K), 0.0416101 secs]
[GC [1 CMS-initial-mark: 80168K(196608K)] 81144K(261184K), 0.0059036 secs]
[CMS-concurrent-mark: 0.129/0.129 secs]
[CMS-concurrent-preclean: 0.007/0.007 secs]
[GC[ Rescan (non-parallel) [ grey object rescan, 0.0020879 secs][root rescan, 0.0144199 secs], 0.016
6258 secs][weak refs processing, 0.0000411 secs] [1 CMS-remark: 80168K(196608K)] 82493K(261184K),
0.0168943 secs]
[CMS-concurrent-sweep: 1.208/1.208 secs]
[CMS-concurrent-reset: 0.036/0.036 secs]
[GC [ParNew: 64576K->960K(64576K), 0.0311520 secs] 66308K->4171K(261184K), 0.0313513 secs]
[GC [ParNew: 64576K->960K(64576K), 0.0348341 secs] 67787K->5695K(261184K), 0.0350776 secs]
[GC [ParNew: 64576K->960K(64576K), 0.0359806 secs] 69311K->7154K(261184K), 0.0362064 secs]
```  

对于CMS日志的详细解读，可以参考这篇[BLOG](http://ivanzhangwb.github.com/blog/2012/12/07/understanding-cms-log/)  

最后，引用一张图来说明Oracle 的JDK6  HotSpot 的 GC 策略

<img src="https://blogs.oracle.com/jonthecollector/resource/Collectors.jpg" width="650px"/>


