---
layout: post
date: 2013-01-04
layout: post
title: "Reids的PUB/SUB"
permalink: "redis-pub-sub"
categories:
- Tools
tags:
- Redis
- NoSql
---



在[来往](http://www.laiwang.com)中有些小地方需要用到一些消息传送的功能，用传统的消息中间件显得很重，一大堆配置不说，还需要编写对应复杂的代码。  

正好在整个系统当中有引入`Redis`，而这个Nosql产品中就有对应的消息订阅与传播功能的实现， 这里简要的介绍下它比较简单的功能， 适用于小规模的消息传播与发送 。

 
`Redis`中针对消息的传播和订阅有两种方式：  

* 针对特定的通道来进行消息传播和订阅
* 针对模式匹配来进行消息传播和订阅 

 其分别对应着三个主要的命令 `PUBLISH`、`SUBSCRIBE` 、`PSUBSCRIBE` 

## PUBLISH 
 
 这个命令的主要作用是发送消息给某个通道或者叫队列中,  
 语法规则为  
 
```
 PUBLISH 通道名 消息内容
```

 比如: 
 
 ```
 PUBLISH TESTCHANNEL MESSAGE1
 ```
 
 表示 往 TESTCHANNEL 这个通道/队列中发送一个消息，消息内容为 MESSAGE1 
 

## SUBSCRIBE

 这个命令的主要作用是订阅某个通道，一旦有消息发过来，它就负责接收消息,  
 语法规则为  
 
```
 SUBSCRIBE 通道名 
```

 比如: 
 
 ```
 SUBSCRIBE TESTCHANNEL 
 ```
 
 表示订阅 TESTCHANNEL 这个通道  。 
 而一旦有消息过来，其就会获取到消息内容，输入则如下所示：  
 
```
1) "message"
2) "通道名"
3) "消息内容"
```  

## PSUBSCRIBE

 这个命令的主要作用是订阅某些符合其给定模式的通道，一旦有消息发过来，它就负责接收消息,  
 语法规则为  
 
```
 PSUBSCRIBE 模式通道名 
```

 比如: 
 
 ```
 PSUBSCRIBE TEST* 
 ```
 
 表示订阅 所有通道名中以TEST开头的通道  。 
 而一旦有消息过来，其就会获取到消息内容，输入则如下所示：  
 
```
1) "pmessage"
2) "模式通道名（比如 TEST*）"
3) "具体通道名"
4) "消息内容"
```  

下图是在我本地的Redis中进行的小例子，简单的用到了这三个命令： 

<img src="http://i01.lw.aliimg.com/21/yj/yj21_5152f732_1218_645.10000x10000x75x2.jpg" width="800px"/>
