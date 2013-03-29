---
date: 2012-01-07
layout: post
title: "Java并发：任务执行"
permalink: '/2012/java-concurrent-task.html'
categories:
- Java
tags:
- java 
- concurrent
---


大多数的并发应用程序都是围绕着执行任务来进行管理的。
任务是对可工作单元的抽象。

用户希望能快速响应。
服务器希望能支持尽可能多的用户使用，每个用户花费的开销越小越好。
当程序负载过高的时候， 程序应该“平缓的恶化”，而不是在负载一高，就简单的以失败告终。
<!-- more -->
过多的/无限的创建线程的缺点：

- 线程生命周期的开销  
   + 创建线程需要时间，带来请求处理的延迟。  
   + 消耗大量计算资源来创建线程，而处理请求的计算资源则被剥夺掉了。

- 资源消耗量
   + 活动线程会引起系统资源消耗，比如内存。  
   + 如果运行的线程数，大于处理器数量，那些多余的线程会闲置起来。  
   + 大量多余的线程占用内存，给GC带来压力。

- 稳定性
   + 过多的创建线程，根据不同的平台可能会产生一些致命的问题，比如OOM，  
   + 任务是逻辑上的单元，而线程是使任务异步化的机制。  
   + 有2种任务执行策略，顺序执行 和 每个任务每个线程。  


### Executor框架
  出现Executor框架后，任务执行的抽象从Thread转换成 Executor.
它是基于生产者和消费者模式， 任务提交是代表着生产待执行的工作单元，
任务的执行表示消费者，它来消耗掉这些工作单元。

#### 关于任务的执行策略
一般来说，将任务提交与任务执行解耦，它的价值在于，让你可以简单地为一个任务制定执行策略。  
一个任务的执行策略一般包括 “3w1h" ---" What,where,when,how"：  

* 任务在什么线程中执行；  
* 任务以什么顺序执行（FIFO,LIFO,优先级)  
* 可以有多少给任务并发执行  
* 有多少任务进入等待队列  
* 系统过载的时候，需放弃任务的时候，如何放弃，放弃哪一个？如何通知程序知道这一切？  
* 一个任务执行结束后，应该做什么处理？

#### 线程池：
线程池一般与工作队列紧密相关，当然，工作队列在线程池的作用，更多的是用来持有所有等待的任务。
而工作者线程则相对轻松很多，从工作队列拿到下一个任务，然后执行它，之后回来，继续等待下一个任务。
线程池的意义在于：
它可以很大程度的重用已经存在的线程，而不需要不断的重新创建线程。

JDK中线程池API：  
`Executors.newFixedThreadPool`  
定长线程池，提交一个任务，创建一个线程，达到池最大程度为止。
`Executors.newCachedThreadPool`
  创建一个可缓存的线程池，当提交的任务过少时，
  池可以灵活的回收线程，当任务大于池的大小时，可以灵活创建线程，不会限制池的大小。
`Executors.newSingleThreadExecutor`  
  创建一个单线程化的executor,只有唯一的线程来处理。

#### Executor的生命周期
ExecutorService扩展了Executor，提供它的生命周期管理。
Executor的生命周期有3种状态：  
  __运行(Running),关闭(shutdown),终止(Terminated)__  
  
- shutdown:
  + 平缓的关闭， 停止接受新任务，同时等待已提交的任务完成。
- shutdownnow:
  + 强制的关闭， 尝试取消所有已经运行中的任务和排在等待队列中尚未开始的任务

在关闭后提交到ExecutorService中的任务，会被拒绝处理器(RejectedException Handler）来处理。  
(JDK内置来四种：`Abort,Discards,DiscardsOldest,CallerRunsPolicy`),  
拒绝处理器可能是简单的放弃一个任务，  
或者抛出一个`RejectedExecutioinException`  
一旦所有任务都执行完成了.Executor会进入到终止状态。

#### 延迟和周期性的任务
Timer类管理着延迟性的任务，或者周期性的任务。
但是它存在一些缺陷，现在可以用ScheduledThreadPoolExecutor来替代它。
（Timer类的一个引起线程泄露的问题， 如果前一个线程抛出异常，则会影响到下一个执行的任务)
 比如： 下面的代码，一旦提交给timer中有一个任务有异常，则后面所有的任务都都不会被执行到了。

	package com.ivanzhang.threadpool;
	import java.util.Timer;
	import java.util.TimerTask;
	public class TimerProblem {
		public static void main(String[] args) throws InterruptedException {
			Timer timer=new Timer();
			timer.schedule(new Mission(), 1);
			Thread.sleep(1000);
			timer.schedule(new Mission(), 1);
			Thread.sleep(5000);
		}
	
	static class Mission extends TimerTask{
		public void run() {
			System.out.println("Start");
			throw new RuntimeException();
		}
	}
	}

输出结果：

	Exception in thread "Timer-0" java.lang.RuntimeException
	at com.ivanzhang.threadpool.TimerProblem$Mission.run(TimerProblem.java:18)
	at java.util.TimerThread.mainLoop(Timer.java:512)
	at java.util.TimerThread.run(Timer.java:462)
	
	Exception in thread "main" java.lang.IllegalStateException: Timer already cancelled.
	at java.util.Timer.sched(Timer.java:354)
	at java.util.Timer.schedule(Timer.java:170)
	at com.ivanzhang.threadpool.TimerProblem.main(TimerProblem.java:11)

携带结果的任务处理：  
利用Callable和Future来处理需要携带结果的任务。  
`ExecutorService.submit` 方法都会返回一个`Future`来抽象一个任务的执行转状况。  

比如一个示例：  
渲染一个网页的时候，通常会有文字和图像，所以可以将工作分成2部来进行；  
一是渲染所有文本，一个是下载所有图像。  
一个是受限于CPU（文本），一个是受限于IO处理。  
我们可以将下载图像抽象成一个任务Callable，提交给ExecutorService来执行。  
然后将渲染文本和下载图像同步进行， 当文本渲染完成后（或者需要图像的时候），  
再去拿下载好的图像，如果幸运的化，我们请求的当时就可以拿到图像了，即使没有，下载可开始了。  
将下载和渲染并行进行。 等所有下载完成后，图片就被呈现在页面上。  
当然，可以优化（ 当用户遇到一个图片的时候（或者下载好一个图片后，就马上显示在页面上）

如果你需要提交一批任务，而且在他们完成后，你要拿到他们所有的状况：  
这个时候，你需要用`CompletionService`.  
它整合了`Executor`和`BlockingQueue`的功能，可以将Callable任务提交给它，  
然后使用类似take和poll  等队列方法，在结果完整可用的时候，获取到结果.  
这样就可以达到上面示例的优化效果，  
使用`CompletionService` 将顺序下载变成 每个下载图片都开始一个独立的线程去做，  
然后提交给`CompletionService`.只要一个下载完成了，就可以马上显示了。  
(completionService.take()拿到一个已经完成的）.  

关于ExecutionCompletionService.take()获取到的Future对象，  
它保证拿到的Future一定是完整可用的，也就是说，通过take()方法获取的Future，  
其timeout没有作用，而且isDone()方法总是为true.

	package com.ivanzhang.threadpool;
	import java.util.ArrayList;
	import java.util.List;
	import java.util.concurrent.Callable;
	import java.util.concurrent.CompletionService;
	import java.util.concurrent.ExecutionException;
	import java.util.concurrent.ExecutorCompletionService;
	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	import java.util.concurrent.Future;
	import java.util.concurrent.TimeUnit;
	import java.util.concurrent.TimeoutException;
	
	public class CompletionServiceDemoSolveProblem {
		public static void main(String[] args) {
			ExecutorService services = Executors.newFixedThreadPool(50);
			List<ImageInfo2> infos = new ArrayList<ImageInfo2>();
			for (int i = 0; i < 15; i++) {
				infos.add(new ImageInfo2(i + "http://www.google.com/images/imageinfo-----" + i + ".jpg"));
			}
	
			CompletionService<String> completionService = new ExecutorCompletionService<String>(
					services);
	
			for (final ImageInfo2 info : infos) {
				completionService.submit(new Callable<String>() {
					@Override
					public String call() throws Exception {
						return info.download();
					}
				});
	
			}
	
			renderText();
	
			for (int i = 0; i < infos.size(); i++) {
				Future<String> result = null;
				String imageId;
				try {
					result = completionService.take();
					imageId = result.get(5, TimeUnit.SECONDS);
				} catch (ExecutionException e) {
					imageId = "default";
					result.cancel(true);
				} catch (TimeoutException e) {
					imageId = "default";
					result.cancel(true);
				} catch (InterruptedException e) {
					imageId = "default";
					result.cancel(true);
				}
				renderImages(imageId);
			}
	
			// 这样，JVM就可以完全退出了.
			services.shutdown();
		}
	
		private static void renderImages(String imagesId) {
			System.out.println("Render Image:[" + imagesId + "]");
		}
	
		private static void renderText() {
			try {
				Thread.sleep(3000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			System.out
					.println("[[[[[RenderText...................................]]]]]]");
	
		}
	}
	
	class ImageInfo2 {
		private String imageId;
		private String url;
	
		public ImageInfo2(String imageId, String url) {
			this.imageId = imageId;
			this.url = url;
		}
	
		public String download() {
			try {
				if (this.imageId.equals("5")) {
					Thread.sleep(5000);
				} else {
					Thread.sleep(2000);
				}
				System.out.println("Download from :" + url + ", Done!!!");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			return imageId;
		}
	}


