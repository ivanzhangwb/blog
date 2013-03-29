---
date: 2012-01-27
layout: post
title: "Apache mod_proxy 与Tomcat7 完成负载均衡"
categories:
- Tools
tags:
- Apache 
- Mod_Proxy
---

使用Apache2.2 来与tomcat7 进行负载均衡。

使用软件：

1.  tomcat7
2.  apache2.2 + mod_proxy模块

第一步： 配置apache mod_proxy
___________________________

将mod_proxy模块加载进去, 具体的配置是修改 httpd.conf文件

	LoadModule proxy_module libexec/apache2/mod_proxy.so
	LoadModule proxy_http_module libexec/apache2/mod_proxy_http.so
	LoadModule proxy_scgi_module libexec/apache2/mod_proxy_scgi.so
	LoadModule proxy_balancer_module libexec/apache2/mod_proxy_balancer.so
	LoadModule proxy_ajp_module libexec/apache2/mod_proxy_ajp.so


第二步： 配置tomcat7
___________________________

为了达到集群的效果， 我们需要在本机启动多个tomcat7的实例。
具体是修改tomcat7 配置目录下面的server.xml文件， 第一个实例的配置文件如下：

	<Server port="9005" shutdown="SHUTDOWN">
	<Connector port="9080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
	<Connector port="9009" protocol="AJP/1.3" redirectPort="8443" />
	<Engine name="Catalina" defaultHost="localhost" jvmRoute="lb2">


第2个配置文件如下：

	<Server port="7005" shutdown="SHUTDOWN">
	<Connector port="7080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
	<Connector port="7009" protocol="AJP/1.3" redirectPort="8443" />
	<Engine name="Catalina" defaultHost="localhost" jvmRoute="lb1">


第三步： 将负载均衡详细配置
___________________________

主要在httpd.conf文件中配置如下代码：

	ProxyRequests Off
	<Proxy balancer://testcluster stickysession=JSESSIONID>
	BalancerMember ajp://127.0.0.1:7009 route=lb1 loadfactor=1
	BalancerMember ajp://127.0.0.1:9009 route=lb2 loadfactor=1
	</Proxy>
	
	ProxyPass / balancer://testcluster/ lbmethod=byrequests stickysession=JSESSIONID

	
下面解释一下，上面代码的具体含义；

	ProxyRequests Off #关闭正向代理，负载均衡就是一个反向代理
	#loadfactor 设置是否平均分配， 以下是1:1
	<Proxy balancer://testcluster stickysession=JSESSIONID>
	BalancerMember ajp://127.0.0.1:7009 route=lb1 loadfactor=1
	BalancerMember ajp://127.0.0.1:9009 route=lb2 loadfactor=1
	</Proxy>
	
	#ProxyPass / balancer://testcluster/ 是协议地址，前后都应该有斜杠 /
	#lbmethod 提供了3种负载方法，分别是 byrequests,bytraffic,bybusyness
	#byrequests : 按照请求次数均衡，是默认的方法
	#bytraffic : 按照流量均衡
	#bybusyness : 按照繁忙程度均衡 （按照活跃请求数量最少的服务器）
	
	ProxyPass / balancer://testcluster/ lbmethod=byrequests stickysession=JSESSIONID


具体示例：
访问 http://localhost/example
则可以看到访问到了tomcat下面的页面了 ， 通过观察tomcat的log文件，具体如下：

	::1 - - [27/Jan/2012:21:13:44 +0800] "GET /examples/ HTTP/1.1" 200 1127
	::1 - - [27/Jan/2012:21:13:46 +0800] "GET /examples/ HTTP/1.1" 304 -
