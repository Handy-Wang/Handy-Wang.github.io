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

* ObjC Crash
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
			![图6](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/ios_app_crash_debug_6.png "图6")
		* Xcode Zoombie法
			* 适用于：EXC_BAD_ACCESS（SIGBUG/SIGSEGV）
			* 修改scheme的配置项
			![图7](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/ios_app_crash_debug_7.png "图7")
		* UncaughtExceptionHandler收集日志
			* 适用于：收集crash日志并上报给服务器
			* [源码](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/UncaughtExceptionHandler.zip)
		* 符号表解析法
			* 适用于：crash日志文件里全是二进制内容，看不出哪里crash的情况下
			* 把.app、.dSYM、.crash三个文件放到一个目录里
			![图8](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/ios_app_crash_debug_8.png "图8")
			* 连上手机，把.crash文件导入到Xcode 的device log里
			![图9](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/ios_app_crash_debug_9.png "图9")
			* Xcode就会desymbol符号表，即可查看导致crash的位置。
		* NSAssert断言跟踪法
			* 适用于：采用NSAssert跟踪代码，即时跟踪到由于服务器脏数据导致的crash
			* 如下：
			![图10](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/ios_app_crash_debug_10.png "图10")
		* 辅助工具：
			* Instruments
			* Crashlytics
			* Bugly
		* 终极大招：注释代码、回滚代码方式的排除法
		* 总结：
			* 分析console输出的crash日志
			* 分析crash trace
			* 分析crash log
			* Crash到main里，建议首先采用全局断点法
			* EXC_BAD_ACCESS时，建议首先采用enable Zombie Objects方法
			* 不要忽视Xcode里的编译警告，可能它们正是问题所在。如果你不明白编译警告的内容，那么就把它弄清楚，因为这会节约你很多不必要的调试程序的时间开销。
			* 注意用真机和模拟器调试时得到的结果可能不一样，建议以真机为准。
			* 正所谓，授之以鱼不如授之以渔，以上都是一些分析的方式，开发中需要按实际情况分析和采用不同的方法。也许以上方法并不能解你遇到的问题，这就需要我们自己趟坑了。
* JS Crash
	* 待续
* 参考文献
	* [my-app-crashed-now-what-part-1](http://www.raywenderlich.com/10209/my-app-crashed-now-what-part-1)
	* [my-app-crashed-now-what-part-2](http://www.raywenderlich.com/10209/my-app-crashed-now-what-part-2)