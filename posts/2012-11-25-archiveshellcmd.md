---
layout: post
date: 2012-11-25 
layout: post
title:"自己常用的*inx命令"
categories:
- Linux
tags:
- Linux
---

常用命令归档
===========


跨平台
------

 `python -m SimpleHTTPServer`   
启动一个简易的web服务器，一般是用于共享文件  

  `echo -n "GET / HTTP/1.0\r\n\r\n" | nc www.baidu.com 80`  
 对一个HTTP服务获取数据
 
 `ls | grep "*mylyn*" | xargs -J % mv % ~/xxxxx`  
 将符合条件的目录和文件，进行备份 (主要是用xargs命令的使用.)
 
 
 
Mac OSX
--------

 `top -u -s5`  
按照CPU使用来对进程进行排序,每隔5秒刷新一次.
<img src="http://i01.lw.aliimg.com/4a/u0/u04a_79f6bfff_1215_172.jpg" width="650px" />

 `top -o cpu -O +rsize -s 5 -n 20`  
 按照CPU和内存排序，每隔5秒刷新一次，最多显示前20个进程信息
 

 `top -orsize -s`   
按照内存大小来排序
<img src="http://i01.lw.aliimg.com/76/u0/u076_7970f894_1216_173.jpg" width="650px" />

 `sysctl -a`  
 查看系统内核参数
<img src="http://i01.lw.aliimg.com/4c/u0/u04c_c0c3824e_803_69.jpg" width="650px" />

 `open .`  
 新开一个finder窗口.  
 `df -h `  
 查看当前磁盘的状态.
<img src="http://i01.lw.aliimg.com/4i/u0/u04i_d846c055_1139_242.jpg" width="650px" />
 `ifconfig | grep cast`  
 查看你的IP地址
<img src="http://i01.lw.aliimg.com/7d/u0/u07d_2d871cc4_873_123.jpg" width="650px" />
 `ioreg -l | grep Capacity `   
查看电池信息
<img src="http://i01.lw.aliimg.com/4o/u0/u04o_13e62780_1119_135.jpg" width="650px" />
  
Linux
------ 
