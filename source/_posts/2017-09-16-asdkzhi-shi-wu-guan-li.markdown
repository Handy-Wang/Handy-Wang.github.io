---
layout: post
title: "ASDK之事务管理"
date: 2017-09-16 20:15:44 +0800
comments: true
categories: 
---

# ASDK之事务管理

ASDK号称异步渲染，其实它的异步渲染就是通过对渲染任务的事务管理来实现，接下来我们就来认识一下。

##### 什么是事务？

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
	
##### ASDK中事务管理的相关类

```text
ASDK中UI渲染操作都是以block的形态抛给事务的(其实不只是UI渲染操作，因为是什么操作完全取决于block里的内容，只是这里我们更关注UI渲染操作)，事务一旦收到这些block任务，就会把它们调度到子线程去执行，
执行的结果会用先暂存起来，待transaction commit时，会把之前暂存的所有结果都刷到界面显示。

所以，请注意ASDK事务以下的几条关键点，它是有别于前面提到的，我们所理解的常规意义的transaction概念的：
1）ASDK事务不只是收集任务，同时还会被立即调度到子线程执行。
2）执行结果被暂存起来。
3）在ASDK事务commit时，只需把之前暂存的结果刷新到界面显示即可。
```

* _ASAsyncTransaction类簇
	* ASAsyncTransactionOperation
		* 别误会，block任务在transaction里至始至终都没有数据结构来封装它们。ASAsyncTransactionOperation是那个暂存执行结果的数据结构，它同时组合了一个回调`block<asyncdisplaykit_async_transaction_operation_completion_block_t>`，用于当transaction commit时，operation把暂存值回传给UI元素进行显示。
	* ASAsyncTransactionGroup
		* 负责在ASDK事务中调度block任务的调度器，它内部会包含一个真正承载block任务的队列。
	* ASAsyncTransactionQueue::Group
		* 调度器管理的的block任务队列，类似于dispatch_group_t
	* ASAsyncTransactionQueue::Operation
		* 
	* _ASAsyncTransaction
		* 接收block任务并调度它们在子线程中执行，各个任务的执行结果存在operation数组中。
* ASAsyncTransactionQueue::Group
* ASAsyncTransactionQueue::GroupImpl
* _ASAsyncTransactionContainer

##### ASDK中事务管理的及时序图

##### 参考资料
* [iOS之让你的App动起来](http://www.jianshu.com/p/f3097f75ede3)
	* 主要参考CATransaction原理、动画原理、渲染原理。
* [Core Animation基本概念和Additive Animation](http://www.cocoachina.com/ios/20140701/8995.html)
	* 主要参考Additive Animation(叠加动画)的概念及它存在的意义。
* [iOS 事件处理机制与图像渲染过程](http://www.cocoachina.com/ios/20151203/14549.html)
	* 主要是看**CA::Transaction::commit();**这一小段，不过其它部分讲得都很精彩。