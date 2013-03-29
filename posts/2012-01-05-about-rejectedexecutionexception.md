---
date: 2012-01-05
layout: post
title: 关于RejectedExecutionException
permalink: '/2012/about-rejectedException.html'
categories:
- Java
tags:
- Java
---


昨天上线一个功能的时候，遇到java.util.concurrent.RejectedExecutionException 这个异常，
   
给系统造成了一定的影响， 记得之前做短信平台项目的时候，有遇到过这个异常，大致知道这个异常的简单意义： 
   
提交的任务被拒绝。
   
<!-- more -->

既然又遇到了，就又仔细的了解了一下这个异常发生的大致原因.
   
目前看来，最主要有2种原因。
1.你的线程池ThreadPoolExecutor 显示的shutdown()之后，再向线程池提交任务的时候。 如果你配置的拒绝策略是AbortPolicy的话，这个异常就会抛出来。
2.当你设置的任务缓存队列过小的时候，或者说， 你的线程池里面所有的线程都在干活（线程数== maxPoolSize),
   
并且你的任务缓存队列也已经充满了等待的队列，  这个时候，你再向它提交任务，则会抛出这个异常。

下面用例子来说明一下，两种情况：

### 第一种， 显示关闭掉了线程池
	
这一点其实理解起来很简单.
	
比如说，你向一个仓库去存放货物，一开始，仓库管理员把门给你打开了，
你放了第一件商品到仓库里面，但是当你放好出去的时候，不小心把仓库的门关掉了，那么你下次再来存放的时候，你就会被拒绝掉。 

落实到代码就是：
{% highlight java %}
ThreadPoolExecutor EXECUTOR = new ThreadPoolExecutor(5, 10, 3000L, 
			TimeUnit.MILLISECONDS,
			
			new LinkedBlockingQueue<Runnable>(4));
	                                              
    for (int i = 0; i < 2; i++) {
        EXECUTOR.execute(new Runnable() {
            public void run() {
                System.out.println("Hello World");
            }
        });
        EXECUTOR.shutdown();
    }
{% endhighlight %}

如上， 在我们提交第一个任务之后，线程池就被关闭掉了。 那么你再向线程池提交新任务的时候，你就会遇到类似的异常。

- **为什么会这样呢？**  
原因是，在ThreadPoolExecutor内部，存放着当前这个线程池的运行状态。  
当你调用shutdown的时候， 线程池会按照顺序将正在运行的任务给关闭掉。
这时候你再像线程池提交任务则会被拒绝掉。

### 第二种情况:
代码如下：
{% highlight java %}
for (int i = 0; i < 15; i++) {
    final int tmpint=i;
    EXECUTOR.execute(new Runnable() {
	        public void run() {
	            try {
	                System.out.println(tmpint+"Hello World");
	                Thread.sleep(1000);
	            } catch (InterruptedException e) {
	                
	            }
	        }
    });
}
{% endhighlight %}

类似的，当你的线程池中 ，正在执行包括正在等待的线程数有 maxPool + workQueueSize 这个数量的话。  
再次向它提交任务，则会遇到这个异常。

### 解决办法： 
针对第一种，暂时还没找打解决方案。 
   
业务上应该尽量避免显示调用shutdown方法。
   
第二种：
   
可以将线程池的数量调大点（minPoolSize,maxPoolSize,handlerList)这新信息。
