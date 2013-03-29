---
date: 2012-01-07
layout: post
title:  "结合JvisualVM的VisualGC插件来学习GC"
permalink: '/2012/jvisualvm-gc-learn.html'
categories:
- MacOs
tags:
- jvisualvm
- jvm
---

学习JAVA的垃圾回收知识有时候会感觉很枯燥，对着命令行往往不够直观。  

这里介绍一款JVISUALVM的插件，它将GC的动作很直观的以图形方式展现出来，  
并将内存的划分区域以不同的颜色标示出来。
<!-- more -->

安装方法：
` 打开JVISUALVM  --- > tool  --> plugin (插件)菜单 ----> aviableble plugin ---> visualGC  `
之后，点击安装即可。
界面如下：  

安装之后，打开某个JAVA进程，定位到visualgc选项卡上，展示的图形类似于：  
![ ](http://farm8.staticflickr.com/7022/6654012569_2ca4561a57_b.jpg)

- - -
如上， 它将内存区域 划分为： 
1.	常量池，
2.	堆内存（Eden区域，2个S区，1给老年区).

看上去非常的直观，再结合GC的一些理论，学习起来会事半功倍。
   
（如图上的标注， 一个对象到达老年区的过程，在下面的图形中很直观的有对应信息查看）
