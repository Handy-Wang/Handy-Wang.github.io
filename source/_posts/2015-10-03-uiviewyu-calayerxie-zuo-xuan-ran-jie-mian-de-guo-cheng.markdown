---
layout: post
title: "UIView与CALayer协作渲染界面的过程"
date: 2015-10-03 01:38:49 +0800
comments: true
categories: 
---
##预热

在聊这个话题前，我先简单的说说与本文相关的几个iOS核心Framework及它们之间的关系：UIKit Framework、QuartzCore Framework(CoreAnimation)、CoreGraphics Framework.

另外，为了能更好的理解本文要讲解的逻辑机制，建议先熟悉一下RunLoop机制，我[在这里](http://blog.handy.wang/blog/2014/05/26/runloopxue-xi-bi-ji-1/)有作讲解。

**注意1**：可以看到上面提到的技术点中的CoreAnimation没有叫 ~~CoreAnimation Framework~~ 。为什么呢？因为CoreAnimation不是被单独地打包到一个Framework里的，而是属于QuartzCore Framework的一部分，奇怪的是QuartzCore.h里只引用了CoreAnimation.h。

      /* QuartzCore.h
	   Copyright (c) 2004-2015, Apple Inc.
	   All rights reserved. */

	#ifndef QUARTZCORE_H
	#define QUARTZCORE_H

	#include <QuartzCore/CoreAnimation.h>

	#endif /* QUARTZCORE_H */

**注意2**：提前了解这几个Framework非常重要，可以辅助我们理解它们各自的职责，以便进一步理解后面会讲解的UIView与CALayer的协作时为什么不同的协作点是相应Framework来参与。

<!-- more -->

![D_UIKit|QuarzCore|CoreGraphics关系图.jpg](http://upload-images.jianshu.io/upload_images/1672953-dbbee8aa6ca50dec.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* **UIKit Framework**
正如Apple官方文档对[UIKit Framework](http://www.baidu.com)的介绍，它主要提供了：界面呈现能力、事件响应能力、驱动RunLoop运行和与系统内核通信的数据。简单来说就是：主要负责界面展示、事件响应以及RunLoop运转动力。*UIView当然是属于UIKit Framework*。

* **QuartzCore Framework 与 CoreAnimation**
正如Apple官方文档对[Quarz Core Framework](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Reference/QuartzCoreRefCollection/)的介绍，它提供了图形处理和视频图像处理的能力。简单来说就是：QuartzCore Framework负责把图形图像最终显示到屏幕上，这里我觉得应该是特指CoreAnimation。不要从字面上去理解CoreAnimation的职责，因为它的核心工作不单是负责动画的创建和执行，它还负责把我们用代码构建的界面显示到屏幕上，实际上是OpenGLES做的。（别急，虽然这里面的机理很复杂，但是在后面会提到）。*CALayer当然是属于QuarzCore Framework下的CoreAnimation*

* **CoreGraphics Framework**
如Apple官方文档对[Core Graphics Framework](https://developer.apple.com/library/prerelease/ios/documentation/CoreGraphics/Reference/CoreGraphics_Framework/index.html)的介绍。CoreGraphics Framework一个基于C库函数的高级绘画引擎，它提供了非常强大的轻量级2D渲染能力。可以使用CoreGraphics处理基于path的绘图工作(如，CGPath)、变形操作(如，CGAffineTransform)、颜色管理(如，CGColor)、离屏渲染(如，CGBitmapContextCreateImage)、渲染模式(patterns)、渐变(gradients)、阴影效果、图形数据管理、图形创建、蒙版以及PDF文档的创建、显示和解析。
**千万别和QuartzCore混淆，CoreGraphics负责创建显示到屏幕上的数据模型，QuartzCore(CoreAnimation -> OpenGLES)负责把CoreGraphics创建的数据模型真正显示到屏幕上。** *CG打头的类都是属于CoreGraphics Framework*

* **以上三者的关系**
三者的关系是通过界面展示以及动画的创建和执行关联起来的，所以它们之间**是协作而不是从属的关系**。在接下来的部分我会从几个方面来阐述以上几个Framework是怎样游游走与UIView与CALayer之间的。

##正文

UIView与CALayer-界面渲染(Rendering)

![D_UIView的显示流程图.jpg](http://upload-images.jianshu.io/upload_images/1672953-bceff4bc5ab8c115.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如图所示，RunLoop/CALayer/UIView之间的协作流程就非常清楚了。下面按图中步骤对流程作下讲解：

1.  目前，我通过代码跟踪总结了两种触发界面渲染的情况
	1. 通过在loadView过程中debug子view的drawRect:方法得知：RunLoop处于kCFRunLoopBeforeWaiting状态时会触发CoreAnimation监听的kCFRunLoopBeforeWaiting的observer，从而通过observer回调来调用CoreAnimation内部的CA::Transaction::commit() ();方法，进而一步一步的调用drawRect方法。
			
			调用流程是：由下至上
			Triggered by BeforeWaiting Observer Callback
	
			#0	0x00000001055825d0 in -[DMQView drawRect:] at /Users/xiaoshan/Home/iOS/SoucesCode/DispatchMainQueue/DispatchMainQueue/DMQView.m:16
			#1	0x0000000106471f08 in -[UIView(CALayerDelegate) drawLayer:inContext:] ()
			#2	0x0000000109f7c183 in -[CALayer drawInContext:] ()
			#3	0x0000000109e7133d in CABackingStoreUpdate_ ()
			#4	0x0000000109f7c002 in ___ZN2CA5Layer8display_Ev_block_invoke ()
			#5	0x0000000109f7be80 in CA::Layer::display_() ()
			#6	0x0000000109f70c69 in CA::Layer::display_if_needed(CA::Transaction*) ()
			#7	0x0000000109f70cf9 in CA::Layer::layout_and_display_if_needed(CA::Transaction*) ()
			#8	0x0000000109f65475 in CA::Context::commit_transaction(CA::Transaction*) ()
			#9	0x0000000109f92c0a in CA::Transaction::commit() ()
	
			#10	0x0000000109f9337c in CA::Transaction::observer_callback(__CFRunLoopObserver*, unsigned long, void*) ()
			#11	0x0000000105f39367 in __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__ ()
			#12	0x0000000105f392d7 in __CFRunLoopDoObservers ()
			#13	0x0000000105f2ef2b in __CFRunLoopRun ()
			#14	0x0000000105f2e828 in CFRunLoopRunSpecific ()
			#15	0x0000000109806ad2 in GSEventRunModal ()
			#16	0x00000001063bb610 in UIApplicationMain ()
			#17	0x00000001055828ef in main at /Users/xiaoshan/Home/iOS/SoucesCode/DispatchMainQueue/DispatchMainQueue/main.m:14
			#18	0x000000010876192d in start ()
			
	2. 通过在VC里添加一个按钮的点击事件并在事件对应的selector中修改子view的背景色，debug子view的drawRect:方法得知：RunLoop被iOS系统传递来的点击事件唤醒并由source1处理(__IOHIDEventSystemClientQueueCallback)，并且进一步由在下一个runloop里的source0转发给UIApplication(_UIApplicationHandleEventQueue)，从而能过source0里的事件队列来调用CoreAnimation内部的CA::Transaction::commit() ();方法，进而一步一步的调用drawRect方法。
	
			调用流程是：由下至上
			Triggered by Source1 [Click Event] -> Source0
	
			#0	0x00000001055825d0 in -[DMQView drawRect:] at /Users/xiaoshan/Home/iOS/SoucesCode/DispatchMainQueue/DispatchMainQueue/DMQView.m:16
			#1	0x0000000106471f08 in -[UIView(CALayerDelegate) drawLayer:inContext:] ()
			#2	0x0000000109f7c183 in -[CALayer drawInContext:] ()
			#3	0x0000000109e7133d in CABackingStoreUpdate_ ()
			#4	0x0000000109f7c002 in ___ZN2CA5Layer8display_Ev_block_invoke ()
			#5	0x0000000109f7be80 in CA::Layer::display_() ()
			#6	0x0000000109f70c69 in CA::Layer::display_if_needed(CA::Transaction*) ()
			#7	0x0000000109f70cf9 in CA::Layer::layout_and_display_if_needed(CA::Transaction*) ()
			#8	0x0000000109f65475 in CA::Context::commit_transaction(CA::Transaction*) ()
			#9	0x0000000109f92c0a in CA::Transaction::commit() ()
	
			#10	0x00000001063b5f7c in _UIApplicationHandleEventQueue ()
			#11	0x0000000105f39a31 in __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ ()
			#12	0x0000000105f2f95c in __CFRunLoopDoSources0 ()
			#13	0x0000000105f2ee13 in __CFRunLoopRun ()
			#14	0x0000000105f2e828 in CFRunLoopRunSpecific ()
			#15	0x0000000109806ad2 in GSEventRunModal ()
			#16	0x00000001063bb610 in UIApplicationMain ()
			#17	0x00000001055828ef in main at /Users/xiaoshan/Home/iOS/SoucesCode/DispatchMainQueue/DispatchMainQueue/main.m:14
			#18	0x000000010876192d in start ()
	3. 可见，上面两种情况都是触发CoreAnimation的CA::Transaction::commit() ();方法来达到触发CALayer/UIView的渲染，所以这个CA::Transaction机制很关键。
2. 其实一步已经进入到了Quarz Core的内部(Core Animation)，即调用CA::Transaction::commit() ();来创建CATrasaction，然后进一步调用-[CALayer drawInContext:] ()
		
		#2	0x0000000109f7c183 in -[CALayer drawInContext:] ()
		#3	0x0000000109e7133d in CABackingStoreUpdate_ ()
		#4	0x0000000109f7c002 in ___ZN2CA5Layer8display_Ev_block_invoke ()
		#5	0x0000000109f7be80 in CA::Layer::display_() ()
		#6	0x0000000109f70c69 in CA::Layer::display_if_needed(CA::Transaction*) ()
		#7	0x0000000109f70cf9 in CA::Layer::layout_and_display_if_needed(CA::Transaction*) ()
		#8	0x0000000109f65475 in CA::Context::commit_transaction(CA::Transaction*) ()
		#9	0x0000000109f92c0a in CA::Transaction::commit() ()
3. 回调CALayer的Delegate(UIView)，问UIView没有需要画的的内容，即回调到drawRect:方法。
4. 在drawRect:方法里可以通过CoreGraphics函数或UIKit中对CoreGraphics封装的方法进行画图操作，这些画图的操作内容都是以Off-Screen离屏(广义的离屏，因为没有在GPU中进行)方式进行画图。[在这里](http://objccn.io/issue-3-1/)可以了解离屏绘图及CPU/GPU的工作。 另，注意图中虚线部分的3|4步骤的情况：因为CALayer可以单独存在进行界面渲染，所以CALayer也可以直接操作CoreGraphics。
5. 无论是有UIView参与的或是直接采用CALayer渲染的操作都会体现在CALayer上(在没有CoreGraphics参与的情况下，UIView或CALayer本身也有一些在业务层面需要显示的内容，所以这里说的“体现在CALayer上”，是泛指UIView或CALayer的子视图子图层以及CoreGraphics参与的内容)。
6. CoreAnimation(CALayer)把它的内容转成位图(纹理)，然后通过OpenGL ES把位图内容传送到GPU的帧缓冲区。
7. 等到由iOS显示屏时钟信号驱动的VSync信号来临时，则把GPU帧缓冲区里的内容显示到iOS显示屏上。[在这里](http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=400417748&idx=1&sn=0c5f6747dd192c5a0eea32bb4650c160&scene=4#wechat_redirect)的**iOS 渲染过程**一节可以了解得更详细。

以上内容就是本章的全部内容，欢迎邮件到nnnwjs@126.com讨论。


## 参考
* Getting Pixels onto the Screen   [英文](https://www.objc.io/issues/3-views/moving-pixels-onto-the-screen/)   |   [中文](http://objccn.io/issue-3-1/)
* [iOS 事件处理机制与图像渲染过程](http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=400417748&idx=1&sn=0c5f6747dd192c5a0eea32bb4650c160&scene=4#wechat_redirect)
* [CFRunLoop](https://github.com/ming1016/study/wiki/CFRunLoop)
* [深入理解RunLoop](http://blog.ibireme.com/2015/05/18/runloop/)
* [线程安全类的设计](http://objccn.io/issue-2-4/)