---
layout: post
title: "浅谈内存布局(Memory Layout)"
date: 2014-03-18 15:14:37 +0800
comments: true
categories: [内存布局, 代码段, 数据段, 堆栈段, C函数调用]
---

对于高级语言的程序员(Java、OC)来说，一定听过堆栈、数据段、代码段。本文在此分享对它们的理解。

<h3>C语言的内存布局</h3>
   这里从C语方入手,说说内存布局。<br/>
{% img https://raw.github.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/Memory-Layout-300x255.gif 400 300 %}<br/>
   内存从低地址到高地址划分，分为：Text Segment、Initialized Data Segment、Uninitialized Data Segment、Heap、Stack。<br/>
   
<h6>Text Segment</h6>
	即代码段(Code Segment)，里面存放可执行的指令，可以理解为程序的实际逻辑代码(不考虑形参、局部变量)，不过在代码段里的指令应该已是汇编指令了。
代码段在内存布局中处在堆栈下面(即低地址段)的位置，防止代码段被堆栈溢出覆盖。通常，代码段里的内容是被共享且只读的，防止程序意外的修改其在代码段里的指令。

<h6>Initialized Data Segment</h6>
	即数据段(Data Segment)，这应该是一种狭义的叫法，也是大家通常意义上认为的数据段。
其实Initialized Data Segment和Uninitialized Data Segment合起来可以称为广义的数据段，因为它们都存数据。
Initialized Data Segment存储被程序员显式初始化过的全局变量、静态变量、常量，数据段不是只读的，所以程序运行的时候可以修改变量的值。
不过，这个狭义的数据段还可以进一步的分为 只读区段 和 可读可写区段。

<h6>Uninitialized Data Segment</h6>
<h6>Heap</h6>
<h6>Stack</h6>


<h3>C++语言的内存布局的差异</h3>
http://www.cnblogs.com/dolphin0520/archive/2011/04/04/2005061.html

其实，没有什么是“自动的”，这只是在搪塞做上层开发的程序员。

<h3>参考</h3>
http://www.geeksforgeeks.org/memory-layout-of-c-program/
http://en.wikipedia.org/wiki/Data_segment
http://en.wikipedia.org/wiki/Call_stack