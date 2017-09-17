---
layout: post
title: "ASDK之事务管理"
date: 2017-09-16 20:15:44 +0800
comments: true
categories: 
---

# ASDK之事务管理

ASDK号称异步渲染，其实它的异步渲染就是通过对渲染任务的事务管理来实现，接下来我们就来认识一下。

* 什么是事务？

	事务是批量操作的容器，在某个时间段内需要执行的操作都可以放到事务里，然后在某个合适的时机一次性完成这些操作。
	
	``` text
	   『数据库事务』
		
		MySQL/Oricle等数据库都是支持事务的。首先，begin一个事务，即建立一个批量操作的容器，然后再进行的批量CRUD操作都被记录到这个事务里，
		最后只需commit事务，数据库引擎就会一次性执行完事务里的所有操作，这样做的好处之一在于只需要一次数据库IO.
		BTW: 数据库事务还支持回滚（当然这是由数据库引擎支撑的），关于事务回滚的内容在这里就不再展开了。
		
	   『iOS事务』
		
		修改View和Layer样式并生效的过程就有事务参与，它就是CATransaction。所有对View和Layer的属性修改或属性动画都被记录到
		CATransaction中，等到RunLoop的状态变更为kCFRunLoopBeforeWaiting时CATransaction就会将自己及子CATransaction中
		记录的属性操作都使其生效到界面上，如果有动画那么生效的过程将是以动画的形式。
		
		实际开发中，修改View和Layer属性或动画操作时基本上没有直接与CATransaction接触，但不带表它不存在，
		RunLoop的最外层就有一个默认的CATransaction，它管理着一个RunLoop周期中对UIView和Layer的属性操作和属性动画。
	```



* ASDK中事务管理的相关类
* ASDK中事务管理的及时序图
* 参考
	* [iOS之让你的App动起来](http://www.jianshu.com/p/f3097f75ede3)
		* 主要参考CATransaction原理、动画原理、渲染原理。
	* [Core Animation基本概念和Additive Animation](http://www.cocoachina.com/ios/20140701/8995.html)
		* 主要参考Additive Animation(叠加动画)的概念及它存在的意义。
	* [iOS 事件处理机制与图像渲染过程](http://www.cocoachina.com/ios/20151203/14549.html)
		* 主要是看**CA::Transaction::commit();**这一小段，不过其它部分讲得都很精彩。