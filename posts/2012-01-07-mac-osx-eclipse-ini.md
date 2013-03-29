---
date: 2012-01-07
layout: post
title: "关于RejectedExecutionException"
permalink: '/2012/mac-eclipse-ini.html'
categories:
- MacOs
tags:
- Mac
- eclipse
---

在windows或者linux下的eclipse，其目录下一般都会有一个eclipse.ini文件，用来配置eclipse的一些属性。Mac OSX下的eclipse配置文件则不在eclipse目录下。
它的路径在：
eclipse目录下/Eclipse.app/Contents/MacOS/eclipse.ini 
这样就可以修改配置文件，比如增加堆内存配置什么的。
<!-- more -->
	-startup
	../../../plugins/org.eclipse.equinox.launcher_1.2.0.v20110502.jar
	--launcher.library
	../../../plugins/org.eclipse.equinox.launcher.cocoa.macosx.x86_64_1.1.100.v20110502
	-product
	org.eclipse.epp.package.java.product
	--launcher.defaultAction
	openFile
	-showsplash
	org.eclipse.platform
	--launcher.XXMaxPermSize
	256m
	--launcher.defaultAction
	openFile
	-vmargs
	-Dosgi.requiredJavaVersion=1.5
	-XstartOnFirstThread
	-Dorg.eclipse.swt.internal.carbon.smallFonts
	-XX:MaxPermSize=256m
	-Xms512m
	-Xmx512m
	-Xdock:icon=../Resources/Eclipse.icns
