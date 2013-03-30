---
layout: post
title: "读《大话处理器》"
date: 2013-03-13 
permalink: "/2013/book/read-about-cpu.html"
categories: 
- Book
---
![大话处理器](http://img3.douban.com/lpic/s8844051.jpg)  

## 指令集
  一般来说，指令集就像是一种语言， 定义一套语言其实并不难，难得是你要让别人去接受你定义的语言。 如果重新使用一套指令集，与之配套的编译器，操作系统、各种应用软件都需要重新编写，这样的工作量和难度，是无法想象的。
  
  从处理器编程模型来看指令集： 比如`C=A+B` , 这个表达式中， `A、B、C`称为`操作数` ，`+`称之为`操作符`. 处理器中做运算的单元称之为`ALU（算术逻辑单元）`。 一般操作数存储在存储器中，而由于存储器中访问数据很慢，因此在离 `ALU`很近的地方放了一些`寄存器（Registers）` 这样，中间计算结果就可以存储在寄存器中，而不用每次都经过存储器了。  
  
  如下图，处理器的运算模型: 
  
  ![](http://i01.lw.aliimg.com/we/yg/ygwe_2846fda1_705_241.680x560x75x2.jpg)
  
 根据处理器的运算模型再回头来看，`C=A+B`这个运算可以看成是一连串的指令，比如 `load R3,#0` 而指令还需要将其编码转换成CPU认识的机器码。 因为计算机只认识 0 和 1， 假设 `load R3,#0`编码后，编程的机器码为 `0101010101` 类似的指令。
 

### 最常见的指令集
 
 *X86*  
 X86是史上最赚钱的指令集，几乎所有个人计算机都是用X86指令集的处理器。 
 1978年，Intel推出了8086，8088处理器，之后，IBM-PC采用8088作为其核心的CPU，随后，Intel又陆续推出几款处理器，比如80286，之后的386、486、奔腾等处理器，X86因此而得名。  
 
 Intel处理器开发有一个非常有名的`tick-tock`战略，也就是时钟的“滴嗒”，在Intel处理器的发展战略上、每一个滴嗒都代表着两年一次的处理器升级。 
 
 而处理器有两大核心技术： 芯片制程工艺、处理器微架构 
 
其余的指令集还有比如： ARM\MIPS\POWER\C6000就不一一介绍了。  
