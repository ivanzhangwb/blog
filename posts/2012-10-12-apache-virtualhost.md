---
date: 2012-10-12
layout: post
title: "Apache Virtualhost"
permalink: "apache-virtual-host"
categories:
- Tools
tags:
- Apache
---

*需求*:   
1. 请求本地的style.china.alibaba.com ,如果本地目录下没有找到，则跳转到另外的IP地址， 主要用途在于,可以快速在本地搭建某个子模块的开发分支， 而不需要将整个应用的代码全部在本地搭建起来.  
2. 同一个IP，用不同的域名访问不同目录下的文件。

*代码*:
	
	NameVirtualHost *:80
	<VirtualHost *:80>
    	DocumentRoot "/Users/zhangwenbo/Code/style"
    	ServerName style.china.alibaba.com
    	<IfModule mod_rewrite.c>
        	RewriteEngine On
        	RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} !-f 
        	RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} !-d 
        	RewriteRule ^(.*)$ http://10.20.136.137$1 [L] 
    	</IfModule>
	</VirtualHost> 
	
	<VirtualHost *:80>
    	DocumentRoot "/Users/zhangwenbo/Code/DocumentRoot"
    	ServerName zwb.com
    	ErrorLog "/Users/zhangwenbo/Code/DocumentRoot/log/error_log"
    	CustomLog "/Users/zhangwenbo/Code/DocumentRoot/log/access_log" common
	</VirtualHost>

*验证结果*:  
1. 将style.china.alibaba.com 和 zwb.com 绑定成本地。 修改/etc/hosts/文件，加入如下代码 `127.0.0.1 localhost zwb.com style.china.alibaba.com`  
2. 访问style.china.alibaba.com 和 zwb.com  可以发现访问的文件已经是不同文件目录下的内容了。
