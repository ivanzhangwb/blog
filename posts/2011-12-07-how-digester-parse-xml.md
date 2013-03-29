---
date: 2011-12-07
layout: post
title: 探求Digester解析XML思路
permalink: '/2011/how-digester-parse-xml.html'
categories:
- OpenSource
tags:
- OpenSource
---


Digester是一个优秀的将XML文件映射到java Bean的开源库，这篇文章旨在探究一个Digester实现常规XML解析的思路。
首先来看看Digester处理XML的示例：
首先是一个XML文件，

	<?xml version="1.0" encoding="gb2312"?>
	<businesses>
		<transport>
		<host>http://localhost</host>
		<port>8080</port>
		<uri>/sms</uri>
		<parameter>bid={0}&amp;amp;amp;origin={1}&amp;amp;amp;receiver={2}&amp;amp;amp;payer={3}&amp;amp;amp;message={4}</parameter>
		</transport>
	</businesses>


我们的目的是希望将这个XML文件转换成我们的Java对象，那么我们一般情况下，会定义一个Java的类，比如下面这样：

	package com.ivanzhangwb.digester;
	import java.util.HashMap;
	import java.util.Map;
	/**
	* 类SmsBussiness.java的实现描述：TODO 类实现描述
	* @author ivanzhangwb 2011-12-7 下午02:15:54
	*/
	public class SmsBussiness {
	private String host;
	private int port;
	private String uri;
	private String parameter;
	Map<String, String> smsBueiness = new HashMap<String, String>();
	
	/**
	* @param host the host to set
	*/
	public void setHost(String host) {
		this.host = host;
	}
	
	/**
	* @param port the port to set
	*/
	public void setPort(int port) {
		this.port = port;
	}
	
	/**
	* @param uri the uri to set
	*/
	public void setUri(String uri) {
		this.uri = uri;
	}
	
	/**
	* @param parameter the parameter to set
	*/
	public void setParameter(String parameter) {
		this.parameter = parameter;
	}
	
	/**
	* @param smsBueiness the smsBueiness to set
	*/
	public void setSmsBueiness(Map<String, String> smsBueiness) {
		this.smsBueiness = smsBueiness;
	}
	
	/* (non-Javadoc)
	* @see java.lang.Object#toString()
	*/
	@Override
	public String toString() {
		return "SmsBussiness [host=" + host + ", port=" + port + ", uri=" + uri + ", parameter=" + parameter
		+ ", smsBueiness=" + smsBueiness + "]";
		}
	}
	

那么，XML文件和对应的Java类都有了，就需要使用Digester来做两者的关系搭建了， 具体代码如下：


	package com.ivanzhangwb.digester;
	import java.io.IOException;
	import org.apache.commons.digester.Digester;
	import org.xml.sax.SAXException;

	public class SmsBussinessRule {
		public static void main(String[] args) throws IOException, SAXException {
			Digester digester = new Digester();
			digester.setValidating(false);
			digester.addObjectCreate("businesses", SmsBussiness.class);
			digester.addBeanPropertySetter("businesses/transport/host", "host");
			digester.addBeanPropertySetter("businesses/transport/port", "port");
			digester.addBeanPropertySetter("businesses/transport/uri", "uri");
			digester.addBeanPropertySetter("businesses/transport/parameter", "parameter");
			SmsBussiness bussiness = (SmsBussiness) digester.parse(SmsBussinessRule.class.getClassLoader().getResourceAsStream("config.xml"));
			System.out.println(bussiness);
		}
	}


  如上，其实我们可以看到，Digester的使用非常简单，它的主要步骤是：
  _初始化  --->  设定规则  ---> 　解析xml文件流_
  但是具体的思路，可以用一张思维导图来表示
  ![Digester解析图](http://farm8.staticflickr.com/7023/6470823989_2372466df6.jpg)

 从上面的图可以看出来， 其Digester处理XML的简单思路：
  * 首先它是集成了SAX来进行XML解析和处理的
  * 它将各种各样的规则归类成它内部的Rule类，来实现各种各样的规则
  * 在其内部，利用SAX的XML事件解析，在接收到解析XML文件的各种事件时，同时再处理它自己的规则(Rule)
 
 放一张简单的时序图：
 
  ![Digester流程](http://farm8.staticflickr.com/7015/6470824223_c6f15b2bfd.jpg)
