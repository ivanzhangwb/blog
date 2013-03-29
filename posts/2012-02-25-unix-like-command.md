--- 
date: 2012-02-25
layout: post
title: "常用Linux实用命令"
categories:
- Linux
tags:
- Linux
---

# 查看java进程中，最占用CPU时间的JAVA线程
 
命令：

	ps -mp pid -o THREAD,tid,time | awk '{print $9"\t"$8}' | sort -r | head -10


- $9: 表示thread id
- $8: 表示占用CPU的时间


通过 man ps 查看帮助信息:

>
>-m Show threads after processes
>-p Select by PID. This selects the processes whose process ID numbers appear in pidlist. 	Identical to p and --pid.
>-o format user-defined format.
>tid 进程下的线程ID(native thread id)

以xxx-web01机器上的java进程为例：

<img src="http://farm8.staticflickr.com/7203/6928680737_dacc9ec761_b.jpg" width="650px"/>

可以看到， 占用CPU最长的线程ID为 4642.
找到了线程ID后，我们需要找到它，为什么会占用这么长时间的CPU，
首先，我们需要将线程ID转换一下，变成成16进制的。（后续会解释，为什么要这么做）


<img src="http://farm8.staticflickr.com/7203/6928681133_5b71b9af5c.jpg" width="650px"/>


接着，我们通过jstack 命令，dump出当前java进程的详细信息。
再从进程的详细信息中，查找我们需要了解的线程信息，
我们利用 jstack pid 查看java进程信息
<img src="http://farm8.staticflickr.com/7190/6782562462_0220a08aaf_b.jpg" width="650px"/>


这里解释一下， 为什么之前，我们需要将线程id转换成 16进制。
因为在jstack命令中输出的详细 java进程中， 会包含有2个thread id 即：tid 和 nid
Tid 和 nid 的区别在于：
Tid 表示的是虚拟机内内存中的线程id
Nid 表示的是操作系统本地的线程id的16进制表示形式值。
而由于之前我们通过ps命令查询到的是 操作系统本地的线程id， 所以我们需要利用nid 的16进制值去查询.

# 查看系统中端口占用的进程信息和相应进程的文件句柄


`lsof –i:80` 查看占用80端口的进程信息
如下: 

<img src="http://farm8.staticflickr.com/7056/6782564220_e4f3db83df_b.jpg" width="650px"/>


查看pid对应的文件句柄有哪些？
<img src="http://farm8.staticflickr.com/7056/6782564220_e4f3db83df_b.jpg" width="650px"/>


可以看到 我机器上的 80端口现在被firefox 占用了， 而firefox引用了一些字体文件和一些程序库文件.
其中有一列为FD， 具体信息如下: 
<img src="http://farm8.staticflickr.com/7059/6782565226_cda45f77bf_b.jpg" width="650px"/>

文件类型如下： man lsof 再搜索 OUTPUT 就可以看到lsof输出的详细信息.

<img src="http://farm8.staticflickr.com/7067/6782566776_29205877d0_b.jpg" width="650px"/>

<img src="http://farm8.staticflickr.com/7208/6782567864_ebd0e66a5e_b.jpg" width="650px"/>

# 查看服务器的IO状态情况

iostat 命令:
<img src="http://farm8.staticflickr.com/7041/6782568396_2755b1783c_b.jpg" width="650px"/>
另外，更多可以参考man 手册和google
