---
layout: post
title: "浅谈内存布局(Memory Layout)"
date: 2014-03-18 15:14:37 +0800
comments: true
categories: [内存布局, 代码段, 数据段, 堆栈段, C函数调用]
---

对于高级语言的程序员(Java、OC)来说，一定听过堆栈、数据段、代码段，但可能没有细细研究过。本文在此分享对它们的理解。
<!--more-->
<h3>C语言的内存布局</h3>
   这里从C语方入手,说说内存布局。<br/>
{% img https://raw.github.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/Memory-Layout-300x255.gif 400 300 %}<br/>
   内存从低地址到高地址划分，分为：Text Segment、Initialized Data Segment、Uninitialized Data Segment、Heap、Stack。<br/>
   
<h6>Text Segment</h6>
	即代码段(Code Segment)，里面存放可执行的指令，可以理解为程序的实际逻辑代码(不考虑形参、局部变量)，不过在代码段里的指令应该已是汇编指令了。
	代码段在内存布局中处在堆栈下面(即低地址段)的位置，防止代码段被堆栈溢出覆盖。通常，代码段里的内容是被共享且只读的，防止程序意外的修改其在代码段里的指令。

<h6>Initialized Data Segment</h6>
	即数据段(Data Segment)，这应该是一种狭义的叫法，也就是大家通常意义上认为的数据段。
	其实Initialized Data Segment和Uninitialized Data Segment合起来可以称为广义的数据段，因为它们都存数据。
	Initialized Data Segment存储被程序员显式初始化过的全局变量、静态变量、常量,由此可以看数据段(狭义)可以进一步的分为 只读区段 和 读写区段，
	数据段不是只读的，在程序运行的时候存储在此区段的变量值可以修改。

<h6>Uninitialized Data Segment</h6>
	又名BSS，即Block Started by Symbol，最早是联合航空公司的联合航空符号汇编程序里的一个伪操作，后来BSS这个术语被合并到FORTRAN汇编程序里。
	这伪操作定义了一个标签而且预留了一定量的未初始化的区域，所以BSS就成了“单独保留许多独立的小数据的位置“的简称。
	由此看出，BSS这部分空间是预留出来的，存放的是程序员没有初始化或初始化为0(我的理解是初始化为nil)的全局变量、静态变量，未初始化的在被编译器在内存中初始化为0(0x0)，
	这就是为什么它也叫Uninitialized Data Segment。
	[Historically, BSS (from Block Started by Symbol) was a pseudo-operation in UA-SAP (United Aircraft Symbolic Assembly Program), 
	the assembler developed in the mid-1950s for the IBM 704 by Roy Nutt, Walter Ramshaw, and others at United Aircraft Corporation.
	The BSS keyword was later incorporated into FAP (FORTRAN Assembly Program), IBM's standard assembler for its 709 and 7090/94 computers. 
	It defined a label (i.e. symbol) and reserved a block of uninitialized space for a given number of words (Timar 1996). 
	In this situation BSS served as a shorthand in place of individually reserving a number of separate smaller data locations. 
	Some assemblers support a complementary or alternative directive BES, for Block Ended by Symbol, 
	where the specified symbol corresponds to the end of the reserved block.]

<h6>Heap</h6>
	堆区紧接着BSS段，它的内存分配增长方向是从低地址向高地址。所有动态的内存分配操作都是由 程序员 通过相关函数从堆区进行分配，如malloc, realloc, and free这些操作。
	在一个进程里，堆区被所有的共享函数库和动态加载的模块所共享。

<h6>Stack</h6>
	栈区紧挨着堆区，一般都处在内存的高地址区域，但是它的内存分配增长方向是从高地址向低地址，也就是说堆和栈的内存分配增长方向上是相对的，即从两头向中间方向。
	此区段的内存分配是由编译器做的，当栈顶指针已经遇到堆指针的时候表明栈溢出了。
	栈区是LIFO的结构，SP(Stack Pointer)寄存器始终记录着栈顶位置，当push或pop数据的时候，SP都会有相应的变化。
	在栈区里主要是存放函数调用相关的内容，如：实参、函数调用的返回地址、局部变量等相关信息，我们称之为栈帧(Stack Frame)，
	也就是说在栈区里有很多个栈帧，每进行一次函数调用，一个新的栈帧就会入栈，且向低地址区方向增长。
	一个栈帧至少由一个函数调用的返回地址组成，所以普通的函数调用递归函数调用都是借助这个返回地址才得到让程序流程进行下去的。
	The stack frame at the top of the stack is for the currently executing routine. The stack frame usually includes at least the following items (in push order):
	the arguments (parameter values) passed to the routine (if any);
	the return address back to the routine's caller (e.g. in the DrawLine stack frame, an address into DrawSquare's code); and
	space for the local variables of the routine (if any).

<h3>与C++语言的内存布局的差异</h3>
	http://www.cnblogs.com/dolphin0520/archive/2011/04/04/2005061.html
待续。。。

<h3>与OC语言的内存布局的差异</h3>
	减少内存垃圾碎片的优化
待续。。。

<h3>C语言函数调用的内存机制</h3>
待续。。。

结束语：
究其细节，其实，没有什么过程是“自动的”，这只不过用来搪塞程序员自己的理由。

<h3>参考</h3>
MemoryLayoutOut: http://www.geeksforgeeks.org/memory-layout-of-c-program/<br/>
DataSegment(wikipedia): http://en.wikipedia.org/wiki/Data_segment<br/>
CallStack(wikipedia): http://en.wikipedia.org/wiki/Call_stack<br/>
C++内存分配、函数调用： http://www.cnblogs.com/dolphin0520/archive/2011/04/04/2005061.html<br/>