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
{% img https://raw.github.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/Memory-Layout-300x255.gif 300 255 %}
   内存从低地址到高地址划分，分为：Text Segment、Initialized Data Segment、Uninitialized Data Segment、Heap、Stack。<br/>


其实，没有什么是“自动的”，这只是在搪塞做上层开发的程序员。