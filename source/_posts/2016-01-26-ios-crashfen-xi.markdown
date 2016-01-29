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
	实际开发中或使用App过程中遇到App crash是一件很常见的事儿，但是要能快速定位是哪一行代码导致的可能就不一定是件易事儿了。在这里我结合自己与别人遇到过的坑，总结了一下。

* 制造一个ObjC crash
	* 代码如下
	![Figure 1-1](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/software_architecture_patterns_figure1_1.png "Figure 1-1")