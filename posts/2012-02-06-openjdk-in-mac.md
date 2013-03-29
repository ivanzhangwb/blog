--- 
date: 2012-02-06
layout: post
title: "自己动手编译 OpenJDK @Mac"
categories:
- JVM
tags:
- OpenJDK
- Mac
---



由于最近研究JVM，所以需要一个自己编译好的JVM来配合学习，下面是在Mac 下编译OpenJDK的过程：

首先需要获取OPENJDK的源代码，需要安装openjdk的源代码管理器--- Mercurial（水银）: 
主页是： <http://mercurial.berkwood.com/>
 
第一步:首先获取 OpenJDK Source
___________________________

	
	hg clone http://hg.openjdk.java.net/macosx-port/macosx-port  macosx-port
	cd macosx-port
	chmod 755 get_source.sh
	./get_source.sh
		
第二步:执行编译
_____________


     make ALLOW_DOWNLOADS=true SA_APPLE_BOOT_JAVA=true ALWAYS_PASS_TEST_GAMMA=true ALT_BOOTDIR=`/usr/libexec/java_home -v 1.6`HOTSPOT_BUILD_JOBS=`sysctl -n hw.ncpu`
     
接着就是等待， 等到编译好以后， 就会在 `macosx-port/build/macosx-universal/bin`

下面执行 `java -version`

	applematoMacBook-Pro:bin apple$ macosx-port/build/macosx-universal/bin/java -version
	openjdk version "1.7.0-internal"
	OpenJDK Runtime Environment (build 1.7.0-internal-apple_2012_02_06_21_19-b00)
	OpenJDK 64-Bit Server VM (build 21.0-b17, mixed mode)

总结
----
 

遇到的问题：

make以后，遇到了乱码问题.

解决办法：

     cd macosx-port
     find build/macosx-universal/corba/gensrc/org/ -name '*.java' | while read p; do native2ascii -encoding UTF-8 $p > tmpj; mv tmpj $p; done
     export _JAVA_OPTIONS=-Dfile.encoding=ASCII
     make ALLOW_DOWNLOADS=true SA_APPLE_BOOT_JAVA=true ALWAYS_PASS_TEST_GAMMA=true ALT_BOOTDIR=`/usr/libexec/java_home -v 1.6` HOTSPOT_BUILD_JOBS=`sysctl -n hw.ncpu`

参考资料：

  [wiki](https://wikis.oracle.com/display/OpenJDK/Mac+OS+X+Port)
  [build-on-mac](http://blog.iusr.me/2011/04/openjdk7-building-on-a-mac/)
