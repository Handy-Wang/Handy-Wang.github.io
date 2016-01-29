---
layout: post
title: "iOS Crash快速分析实战"
date: 2016-01-26 10:57:50 +0800
comments: true
tags: [iOS, App Crash, Debug]
keywords: iOS App Crash, Debug
description: 快速定位导致App crash的代码
categories: 
---
	实际开发中或使用App过程中遇到App crash是一件很常见的事儿，但是要能快速定位是哪一行代码导致的可能就不一定是件易事儿了。
	在这里我结合自己与别人遇到过的坑，总结了一下。
	
<!--more-->

* ObjC crash
	* 制造一个Crash，代码如下
	![图1](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/ios_app_crash_debug_1.png "图1")
	
	* Crash了，假装不知道是由于什么导致的：
	![图2](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/ios_app_crash_debug_2.png "图2")
	
	* 左侧的Crash trace 也是很让人无语
	![图3](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/ios_app_crash_debug_3.png "图3")
	
	* 展开看一下更详细的crash trace
	![图4](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/ios_app_crash_debug_4.png "图4")
	
	* 想办法排查：
		* 全局断点法（All Exceptions）
			* 适用于：停留到main里的crash，可能是EXC_CRASH（SIGABRT）或EXC_BAD_ACCESS（SIGBUG/SIGSEGV）导致的
			* 加上全局断点再run，就可以看见哪一行导致crash了。
	![图5](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/ios_app_crash_debug_5.png "图5")